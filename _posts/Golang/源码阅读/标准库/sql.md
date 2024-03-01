// conn 返回一个新的或者缓存的连接
func (db *DB) conn(ctx context.Context, strategy connReuseStrategy) (*driverConn, error) {
   db.mu.Lock()
   if db.closed {
      db.mu.Unlock()
      return nil, errDBClosed
   }
   // Check if the context is expired.
   select {
   default:
   case <-ctx.Done():
      db.mu.Unlock()
      return nil, ctx.Err()
   }
   lifetime := db.maxLifetime

   // Prefer a free connection, if possible.
   numFree := len(db.freeConn)
   if strategy == cachedOrNewConn && numFree > 0 {
      conn := db.freeConn[0]
      copy(db.freeConn, db.freeConn[1:])
      db.freeConn = db.freeConn[:numFree-1]
      conn.inUse = true
      if conn.expired(lifetime) {
         db.maxLifetimeClosed++
         db.mu.Unlock()
         conn.Close()
         return nil, driver.ErrBadConn
      }
      db.mu.Unlock()

      // Reset the session if required.
      if err := conn.resetSession(ctx); err == driver.ErrBadConn {
         conn.Close()
         return nil, driver.ErrBadConn
      }
    
      return conn, nil
   }

   // Out of free connections or we were asked not to use one. If we're not
   // allowed to open any more connections, make a request and wait.
   if db.maxOpen > 0 && db.numOpen >= db.maxOpen {
      // Make the connRequest channel. It's buffered so that the
      // connectionOpener doesn't block while waiting for the req to be read.
      req := make(chan connRequest, 1)
      reqKey := db.nextRequestKeyLocked()
      db.connRequests[reqKey] = req
      db.waitCount++
      db.mu.Unlock()

      waitStart := nowFunc()
    
      // Timeout the connection request with the context.
      select {
      case <-ctx.Done():
         // Remove the connection request and ensure no value has been sent
         // on it after removing.
         db.mu.Lock()
         delete(db.connRequests, reqKey)
         db.mu.Unlock()
    
         atomic.AddInt64(&db.waitDuration, int64(time.Since(waitStart)))
    
         select {
         default:
         case ret, ok := <-req:
            if ok && ret.conn != nil {
               db.putConn(ret.conn, ret.err, false)
            }
         }
         return nil, ctx.Err()
      case ret, ok := <-req:
         atomic.AddInt64(&db.waitDuration, int64(time.Since(waitStart)))
    
         if !ok {
            return nil, errDBClosed
         }
         // Only check if the connection is expired if the strategy is cachedOrNewConns.
         // If we require a new connection, just re-use the connection without looking
         // at the expiry time. If it is expired, it will be checked when it is placed
         // back into the connection pool.
         // This prioritizes giving a valid connection to a client over the exact connection
         // lifetime, which could expire exactly after this point anyway.
         if strategy == cachedOrNewConn && ret.err == nil && ret.conn.expired(lifetime) {
            db.mu.Lock()
            db.maxLifetimeClosed++
            db.mu.Unlock()
            ret.conn.Close()
            return nil, driver.ErrBadConn
         }
         if ret.conn == nil {
            return nil, ret.err
         }
    
         // Reset the session if required.
         if err := ret.conn.resetSession(ctx); err == driver.ErrBadConn {
            ret.conn.Close()
            return nil, driver.ErrBadConn
         }
         return ret.conn, ret.err
      }
   }

   db.numOpen++ // optimistically
   db.mu.Unlock()
   ci, err := db.connector.Connect(ctx)
   if err != nil {
      db.mu.Lock()
      db.numOpen-- // correct for earlier optimism
      db.maybeOpenNewConnections()
      db.mu.Unlock()
      return nil, err
   }
   db.mu.Lock()
   dc := &driverConn{
      db:         db,
      createdAt:  nowFunc(),
      returnedAt: nowFunc(),
      ci:         ci,
      inUse:      true,
   }
   db.addDepLocked(dc, dc)
   db.mu.Unlock()
   return dc, nil
}