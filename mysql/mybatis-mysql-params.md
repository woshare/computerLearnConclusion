# Mybatis & mysql 中参数介绍

## mybatis的poolMaximumActiveConnections，poolMaximumIdleConnections和Mysql的max_connections bug问题
>1,mybatis中PooledDataSource的默认参数连接池最大活跃数 poolMaximumActiveConnections 默认10 ，链接池最大空闲数poolMaximumIdleConnections默认5 
>2,mysql的参数max_connections=200 
>3，最后mysql返回"too many connection"
>4，原因：PooledDataSource.pushConnection 中只要空闲数少于最大，则又会创建链接，从而超过mysql的最大连接数限制
>5，解决办法：poolMaximumIdleConnections=0
```
protected void pushConnection(PooledConnection conn) throws SQLException {

    synchronized (state) {
      state.activeConnections.remove(conn);
      if (conn.isValid()) {
        if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {
          state.accumulatedCheckoutTime += conn.getCheckoutTime();
          if (!conn.getRealConnection().getAutoCommit()) {
            conn.getRealConnection().rollback();
          }
          PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
          state.idleConnections.add(newConn);
          newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
          newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());
          conn.invalidate();
          if (log.isDebugEnabled()) {
            log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");
          }
          state.notifyAll();
        } else {
          state.accumulatedCheckoutTime += conn.getCheckoutTime();
          if (!conn.getRealConnection().getAutoCommit()) {
            conn.getRealConnection().rollback();
          }
          conn.getRealConnection().close();
          if (log.isDebugEnabled()) {
            log.debug("Closed connection " + conn.getRealHashCode() + ".");
          }
          conn.invalidate();
        }
      } else {
        if (log.isDebugEnabled()) {
          log.debug("A bad connection (" + conn.getRealHashCode() + ") attempted to return to the pool, discarding connection.");
        }
        state.badConnectionCount++;
      }
    }
  }


private PooledConnection popConnection(String username, String password) throws SQLException {
    boolean countedWait = false;
    PooledConnection conn = null;
    long t = System.currentTimeMillis();
    int localBadConnectionCount = 0;

    while (conn == null) {
      synchronized (state) {
        if (!state.idleConnections.isEmpty()) {
          // Pool has available connection
          conn = state.idleConnections.remove(0);
          if (log.isDebugEnabled()) {
            log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
          }
        } else {
          // Pool does not have available connection
          if (state.activeConnections.size() < poolMaximumActiveConnections) {
            // Can create new connection
            conn = new PooledConnection(dataSource.getConnection(), this);
            if (log.isDebugEnabled()) {
              log.debug("Created connection " + conn.getRealHashCode() + ".");
            }
          } else {
            // Cannot create new connection
            PooledConnection oldestActiveConnection = state.activeConnections.get(0);
            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
            if (longestCheckoutTime > poolMaximumCheckoutTime) {
              // Can claim overdue connection
              state.claimedOverdueConnectionCount++;
              state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
              state.accumulatedCheckoutTime += longestCheckoutTime;
              state.activeConnections.remove(oldestActiveConnection);
              if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                try {
                  oldestActiveConnection.getRealConnection().rollback();
                } catch (SQLException e) {
                  /*
                     Just log a message for debug and continue to execute the following
                     statement like nothing happened.
                     Wrap the bad connection with a new PooledConnection, this will help
                     to not interrupt current executing thread and give current thread a
                     chance to join the next competition for another valid/good database
                     connection. At the end of this loop, bad {@link @conn} will be set as null.
                   */
                  log.debug("Bad connection. Could not roll back");
                }
              }
              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
              conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
              conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
              oldestActiveConnection.invalidate();
              if (log.isDebugEnabled()) {
                log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
              }
            } else {
              // Must wait
              try {
                if (!countedWait) {
                  state.hadToWaitCount++;
                  countedWait = true;
                }
                if (log.isDebugEnabled()) {
                  log.debug("Waiting as long as " + poolTimeToWait + " milliseconds for connection.");
                }
                long wt = System.currentTimeMillis();
                state.wait(poolTimeToWait);
                state.accumulatedWaitTime += System.currentTimeMillis() - wt;
              } catch (InterruptedException e) {
                break;
              }
            }
          }
        }
        if (conn != null) {
          // ping to server and check the connection is valid or not
          if (conn.isValid()) {
            if (!conn.getRealConnection().getAutoCommit()) {
              conn.getRealConnection().rollback();
            }
            conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));
            conn.setCheckoutTimestamp(System.currentTimeMillis());
            conn.setLastUsedTimestamp(System.currentTimeMillis());
            state.activeConnections.add(conn);
            state.requestCount++;
            state.accumulatedRequestTime += System.currentTimeMillis() - t;
          } else {
            if (log.isDebugEnabled()) {
              log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
            }
            state.badConnectionCount++;
            localBadConnectionCount++;
            conn = null;
            if (localBadConnectionCount > (poolMaximumIdleConnections + poolMaximumLocalBadConnectionTolerance)) {
              if (log.isDebugEnabled()) {
                log.debug("PooledDataSource: Could not get a good connection to the database.");
              }
              throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
            }
          }
        }
      }

    }

    if (conn == null) {
      if (log.isDebugEnabled()) {
        log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
      }
      throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
    }

    return conn;
  }

```


* [poolMaximumActiveConnections，poolMaximumIdleConnections和Mysql的max_connections bug问题](https://my.oschina.net/sniperLi/blog/550680)


## JDBC 中常用参数
>1，UsePerformanceMonitor，userperfmon, perfmon：是否启用性能监视，默认 false
>2，IgnorePrepare：    是否忽略 Prepare() 调用，默认 true
>3，UseProcedureBodies,procedure bodies：是否检查存储过程体、参数的有效性，默认 true
>4，AutoEnlist：    是否自动使用活动的连接，默认 true
>5，TreatTinyAsBoolean：是否将 TINYINT(1) 列视为布尔型，默认 true
>6，AllowUserVariables：是否允许 SQL 中出现用户变量，默认 false
>7，FunctionsReturnString：所有服务器函数是否按返回字符串处理，默认 false
>8，UseAffectedRows：是否用受影响的行数替代查找到的行数来返回数据，默认 false
>9，Keepalive：    保持 TCP 连接的秒数，默认0，不保持。
>10，ConnectionLifeTime：连接被销毁前在连接池中保持的最少时间（秒）。默认 0
>11，Pooling：    是否使用线程池，默认 true
>12，MinimumPoolSize, min pool size：线程池中允许的最少线程数，默认 0
>13，MaximumPoolSize，max pool size：线程池中允许的最多线程数，默认 100
>14，ConnectionReset：连接过期后是否自动复位，默认 false
>15，CharacterSet, charset：向服务器请求连接所使用的字符集，默认：无