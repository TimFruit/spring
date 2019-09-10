

// Spring 开启事务之后, 即从数据源获取了connection之后, 会将connection放进TransactionSynchronizationManager管理
// 内部是使用ThreadLocal保存, 即与当前线程以及datasource绑定 --> ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal<>("Transactional resources");
// mybaits使用执行器(Executor)执行时, 会委派给SpringManagedTransaction获取connection , 
// 而该Transaction是通过DataSourceUtils从TransactionSynchronizationManager获取key为datasource的资源(connection)
// 
// spring和mybatis通过TransactionSynchronizationManager交流, 获取同一对应的connection, 进而合作完成事务
// spring通过connection完成了事务的开启 提交或者回滚操作
// 而mybatis则使用connection完成事务中sql语句的执行

1. SpringManagedTransaction实现了mybatis的Transaction接口
    SpringManagedTransactionFactory实现了mybatis的TransactionFactory接口, 用于生成SpringManagedTransaction
    
    事务是如何交给spring管理的呢?
    查看SpringManagedTransaction可知, 使用工具类DataSourceUtils管理事务
    ```
     /**
       * {@inheritDoc}
       */
      @Override
      public Connection getConnection() throws SQLException {
        if (this.connection == null) {
          openConnection();
        }
        return this.connection;
      }
    
    
     @Override
     public void close() {
       DataSourceUtils.releaseConnection(this.connection, this.dataSource);
     }
        
      /**
       * Gets a connection from Spring transaction manager and discovers if this
       * {@code Transaction} should manage connection or let it to Spring.
       * <p>
       * It also reads autocommit setting because when using Spring Transaction MyBatis
       * thinks that autocommit is always false and will always call commit/rollback
       * so we need to no-op that calls.
       */
      private void openConnection() throws SQLException {
        this.connection = DataSourceUtils.getConnection(this.dataSource);
        this.autoCommit = this.connection.getAutoCommit();
        this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);
    
        LOGGER.debug(() ->
            "JDBC Connection ["
                + this.connection
                + "] will"
                + (this.isConnectionTransactional ? " " : " not ")
                + "be managed by Spring");
      }
    ```
    
    //
    //
    DataSourceUtils
    ```
    public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
            try {
                return doGetConnection(dataSource);
            } catch (SQLException var2) {
                throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection", var2);
            } catch (IllegalStateException var3) {
                throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection: " + var3.getMessage());
            }
        }
    
    public static Connection doGetConnection(DataSource dataSource) throws SQLException {
        Assert.notNull(dataSource, "No DataSource specified");
        // spring使用事务同步管理器来管理事务资源
        ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
        if (conHolder == null || !conHolder.hasConnection() && !conHolder.isSynchronizedWithTransaction()) {
            logger.debug("Fetching JDBC Connection from DataSource");
            Connection con = fetchConnection(dataSource);
            if (TransactionSynchronizationManager.isSynchronizationActive()) {
                try {
                    ConnectionHolder holderToUse = conHolder;
                    if (conHolder == null) {
                        holderToUse = new ConnectionHolder(con);
                    } else {
                        conHolder.setConnection(con);
                    }

                    holderToUse.requested();
                    TransactionSynchronizationManager.registerSynchronization(new DataSourceUtils.ConnectionSynchronization(holderToUse, dataSource));
                    holderToUse.setSynchronizedWithTransaction(true);
                    if (holderToUse != conHolder) {
                        TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
                    }
                } catch (RuntimeException var4) {
                    releaseConnection(con, dataSource);
                    throw var4;
                }
            }

            return con;
        } else {
            conHolder.requested();
            if (!conHolder.hasConnection()) {
                logger.debug("Fetching resumed JDBC Connection from DataSource");
                conHolder.setConnection(fetchConnection(dataSource));
            }

            return conHolder.getConnection();
        }
    }
    ```



