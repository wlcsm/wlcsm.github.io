---
title: "Using Errgroup"
date: 2021-10-01
tags: ["golang"]
---

Here I'm going to illustrate a couple of common use case for the errgroup, and in particular some in which you may think the errgroup is too fundamental but is in fact not.

# Run many things in parallel and return if there is an error or a certain response

For instance, multiple validation functions, and immediately return if an error is returned or a function failed. Or to poll multiple data centers for a result and immediately return the first one.

If the functions return an error that this is no problem, but if the user just doesn't have permissions, then at this abstraction layer, this probably shouldn't be considered an error, and the function should just return false. So here we are going to assume the validation functions have the type synature func(ctx context.Context) (bool, error)

```go
func concurrentValidate(ctx context.Context, funcs []func(ctx context.Context) (bool, error)) (bool, error) {
	hasAuth := true
	var setHasAuth sync.Once

	ctx, cancel := context.WithCancel(ctx)
	errs, ctx := errgroup.WithContext(ctx)
	for _, fi := range funcs {
		f := fi
		errs.Go(func() error {
			ok, err := f(ctx)
			if err {
				return Wrap(err)
			}
			if !ok {
				cancel()
				setHasAuth.Do(func() { hasAuth = false })
			}
			return nil
		}
	}

	if err := errs.Wait(); err != nil {
		return false, Wrap(err)
	}

	return hasAuth, nil
```

Aggregate many elements into a list. Sure you can do this with a channel, but thats not important. The important thing is that this gives a fairly straightforward way to do things

```go
func concurrentAggregate(ctx context.Context, funcs []func(ctx context.Context) (interface{}, error)) ([]interface{}, error) {
	res := make([]interface{}, 0)
	var mu sync.Mutex

	errs, ctx := errgroup.WithContext(ctx)
	for _, fi := range funcs {
		f := fi
		errs.Go(func() error {
			v, err := f(ctx)
			if err {
				return Wrap(err)
			}

			mu.Lock()
			res = append(res, v)
			mu.Unlock()

			return nil
		}
	}

	if err := errs.Wait(); err != nil {
		return nil, Wrap(err)
	}

	return res, nil
```

Since Go's load balancer is pretty good and goroutine's footprint is quite small, there is often no need to limit the number of goroutines spawned by the errgroup. If you really need it you can probably get away with a simple semaphore

```go
func concurrentAggregate(ctx context.Context, funcs []func(ctx context.Context) (interface{}, error)) ([]interface{}, error) {
	sema := semaphore.WithLimit(1)

	ctx, cancel := context.WithCancel(ctx)
	errs, ctx := errgroup.WithContext(ctx)
	for _, fi := range funcs {
		f := fi

		if err := sema.Acquire(ctx, 1); err != nil {
			cancel()
			return err
		}

		errs.Go(func() error {
			defer sema.Release()

			// your code
		}
	}

	if err := errs.Wait(); err != nil {
		return nil, Wrap(err)
	}

	return res, nil
```

