# Kubernetes leader election data race debugging code

```
go build -race && GORACE="halt_on_error=1" ./leader-race
```

## Found races

```
==================
WARNING: DATA RACE
Write at 0x00c0002e9de8 by goroutine 36:
  k8s.io/client-go/tools/leaderelection/resourcelock.(*EndpointsLock).Get()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/resourcelock/endpointslock.go:42 +0x208
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).tryAcquireOrRenew()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:327 +0x18f
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1.1.1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:268 +0x6c

Previous read at 0x00c0002e9de8 by main goroutine:
  k8s.io/client-go/tools/leaderelection/resourcelock.(*EndpointsLock).RecordEvent()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/resourcelock/endpointslock.go:95 +0x1df
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:285 +0x261
  k8s.io/apimachinery/pkg/util/wait.JitterUntil.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:152 +0x6f
  k8s.io/apimachinery/pkg/util/wait.JitterUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:153 +0x108
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:88 +0x16d
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).Run()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:207 +0x188
  main.main()
      /Users/cezarsa/code/leader-race/main.go:59 +0x398

Goroutine 36 (running) created at:
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1.1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:266 +0xa9
  k8s.io/apimachinery/pkg/util/wait.PollImmediateUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:393 +0x38
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:264 +0x1ab
  k8s.io/apimachinery/pkg/util/wait.JitterUntil.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:152 +0x6f
  k8s.io/apimachinery/pkg/util/wait.JitterUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:153 +0x108
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:88 +0x16d
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).Run()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:207 +0x188
  main.main()
      /Users/cezarsa/code/leader-race/main.go:59 +0x398
==================
```

```
==================
WARNING: DATA RACE
Read at 0x00c000322720 by main goroutine:
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).maybeReportTransition()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:374 +0x47
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:279 +0x1d3
  k8s.io/apimachinery/pkg/util/wait.JitterUntil.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:152 +0x6f
  k8s.io/apimachinery/pkg/util/wait.JitterUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:153 +0x108
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:88 +0x16d
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).Run()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:207 +0x188
  main.main()
      /Users/cezarsa/code/leader-race/main.go:59 +0x398

Previous write at 0x00c000322720 by goroutine 17:
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).tryAcquireOrRenew()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:344 +0xd4d
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1.1.1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:268 +0x6c

Goroutine 17 (running) created at:
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1.1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:266 +0xa9
  k8s.io/apimachinery/pkg/util/wait.PollImmediateUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:393 +0x38
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:264 +0x1ab
  k8s.io/apimachinery/pkg/util/wait.JitterUntil.func1()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:152 +0x6f
  k8s.io/apimachinery/pkg/util/wait.JitterUntil()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:153 +0x108
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).renew()
      /Users/cezarsa/go/pkg/mod/k8s.io/apimachinery@v0.0.0-20190923155427-ec87dd743e08/pkg/util/wait/wait.go:88 +0x16d
  k8s.io/client-go/tools/leaderelection.(*LeaderElector).Run()
      /Users/cezarsa/go/pkg/mod/k8s.io/client-go@v0.0.0-20190924155751-0df3af31cdd4/tools/leaderelection/leaderelection.go:207 +0x188
  main.main()
      /Users/cezarsa/code/leader-race/main.go:59 +0x398
==================
```