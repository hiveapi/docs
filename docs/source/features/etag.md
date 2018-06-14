# ETags

HiveApi provides an ETag Middleware (located in `app/Ship/Middlewares/Http/ProcessETagHeadersMiddleware.php`) that 
implements the shallow technique. It can be used to reduce bandwidth on the client side (especially useful for mobile 
devices like smartphones or tablets).

By default the feature is disabled. To enable it go to `app/Ship/Configs/hive.php` configuration file and set `use-etag` 
to `true`. Of course your client should send the `If-None-Match` HTTP Header `(= etag)` in all requests for this feature 
to work properly.
