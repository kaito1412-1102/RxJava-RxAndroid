# RxJava-RxAndroid

*Disposable: 
    private CompositeDisposable disposable = new CompositeDisposable();

Disposable được sử dụng để hủy sự kết nối của Subserver với Subsevable khi không còn cần thiết việc này rất hữu dụng để tránh việc rò rỉ bộ nhớ.
Khi Observer kết nối được với Observable trong onSubcribe() ta sẽ nhận được Disposable. Để hủy sự kết nối trong onDestroy() của Activity bạn nên gọi hàm dispose() của Disposable.


*fromIterable:

Observable<Task> taskObservable = Observable.fromIterable(DataSource.createTasksList())
                .subscribeOn(Schedulers.io())
                .filter(new Predicate<Task>() {
                    @Override
                    public boolean test(Task task) throws Throwable {
                        return task.isComplete();
                    }
                });

        taskObservable.subscribe(new Observer<Task>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
              disposable.add(d);
            }

            @Override
            public void onNext(@NonNull Task task) {
                Log.d(TAG, "Thread: " + Thread.currentThread().getName());
                Log.d(TAG, "onNext: " + task.getDescription());
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
  
  
  *Observable.defer:  khi bạn có một cái gì đó tạo / trả về một cái có thể quan sát được, nhưng bạn không muốn quá trình đó xảy ra cho đến khi đăng ký.
  
  Observable<Task> a = Observable.defer(new Supplier<ObservableSource<? extends Task>>() {
            @Override
            public ObservableSource<? extends Task> get() throws Throwable {
                return Observable.fromIterable(DataSource.createTasksList());
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread());
        a.subscribe(new Observer<Task>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Task task) {
                Log.d(TAG, "onNext1: " + task.getDescription());
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
  
  
