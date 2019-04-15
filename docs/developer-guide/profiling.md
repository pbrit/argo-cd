# Profiling

## Finding goroutine Leaks

Apply this patch:

```diff
diff --git a/server/server.go b/server/server.go
index cc73fa3..7032b40 100644
--- a/server/server.go
+++ b/server/server.go
@@ -7,6 +7,7 @@ import (
        "io/ioutil"
        "net"
        "net/http"
+       _ "net/http/pprof"
        "net/url"
        "os"
        "os/exec"
@@ -258,6 +259,12 @@ func (a *ArgoCDServer) Run(ctx context.Context, port int) {
                log.Fatal("Timed out waiting for project cache to sync")
        }
 
+       go func() {
+               err := http.ListenAndServe(":6060", nil)
+               if err != nil {
+                       log.Fatal(err)
+               }
+       }()
        a.stopCh = make(chan struct{})
        <-a.stopCh
        errors.CheckError(conn.Close())
```

Start up it up:

```bash
make start
```

Run the E2E test several times:

```bash
for i in $(seq 1 10) ; do make test-e2e ; done
```

Open the profile in the tool, maybe create a PNG of the leaks:

```bash
go tool pprof http://localhost:6060/debug/pprof/goroutine
$ png
```

Maybe check the [debug output](http://localhost:6060/debug/pprof/goroutine?debug=1).