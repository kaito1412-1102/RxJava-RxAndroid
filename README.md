# RxJava-RxAndroid

doc:    https://viblo.asia/p/cung-hoc-rxjava-phan-1-gioi-thieu-aRBeXWqgGWE
        https://viblo.asia/p/cung-hoc-rxjava-phan-2-threading-concept-MgNeWWwXeYx
        https://viblo.asia/p/cung-hoc-rxjava-phan-3-core-operators-mrDkMrpzvzL

*Disposable: 
    private CompositeDisposable disposable = new CompositeDisposable();

Disposable được sử dụng để hủy sự kết nối của Subserver với Subsevable khi không còn cần thiết việc này rất hữu dụng để tránh việc rò rỉ bộ nhớ.
Khi Observer kết nối được với Observable trong onSubcribe() ta sẽ nhận được Disposable. Để hủy sự kết nối trong onDestroy() của Activity bạn nên gọi hàm dispose() của Disposable.

*from: 
    
    Observable.from(1,2,3).subscribe(new Subscriber<Integer>() {
            public void onCompleted() {

            }

            public void onError(Throwable e) {

            }

            public void onNext(Integer integer) {
               Log.i("onNext", String.valueOf(integer));
            }
        });

*just: 
    
    Observable.just(1,2,3).subscribe(new Subscriber<Integer>() {
            public void onCompleted() {

            }

            public void onError(Throwable e) {

            }

            public void onNext(Integer integer) {
               Log.i("onNext", String.valueOf(integer));
            }
        });

just và from khác nhau như nào?

    Integer[] integers = {1,2,3};    //Danh sách

    Observable.just(integers).subscribe(new Subscriber<Integer[]>() {
        public void onNext(Integer[] integers) {            //Phát ra cả 1 danh sách
            Log.i("onNext", Arrays.toString(integers));
        }
    }

    Observable.from(integers).subscribe(new Subscriber<Integer>() {
        public void onNext(Integer integer) {               //Phát ra từng item trong danh sách
            Log.i("onNext", String.valueOf(integer));
        }
    }
    
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
  
*Observable.create: 

    Task task = new Task("Walk the dog", false, 3);
    
        Observable<Task> taskObservable = Observable.create(new ObservableOnSubscribe<Task>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Task> emitter) throws Throwable {
                if(!emitter.isDisposed()){
                    emitter.onNext(task);
                    emitter.onComplete();
                }
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread());
        taskObservable.subscribe(new Observer<Task>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Task task) {
                Log.d(TAG, "onNext: " + task.getDescription());
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
  
*Observable.range: Toán tử range phát ra một phạm vi các số nguyên tuần tự, theo thứ tự, nơi bạn chọn đầu của phạm vi và độ dài của phạm vi đó.
    
    Observable<Integer> observable = Observable
                .range(0, 11)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread());
        observable.subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Integer integer) {
                Log.d(TAG, "onNext: " + integer);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
        
*Observable.interval:   sẽ phát ra 1 item thuộc kiểu Long sau mỗi khoảng delay.

    Observable.interval(2, TimeUnit.SECONDS).subscribe(new Subscriber<Long>() {
            public void onNext(Long aLong) {
                if(aLong == 5)
                    unsubscribe();
                Log.i("onNext", "" + aLong);
            }
        });
        
        
Dễ thấy rằng mặc định Rx chỉ chạy trên thread mà hàm subscribe() được gọi đến. Tuy có bản chất là mô hình luồng tự do nhưng không có nghĩa rằng Rx sẽ tự động sử dụng đa luồng cho bạn, nó chỉ mang ý nghĩa là bạn có thể chọn bất cứ thread nào để thực thi công việc trên đó.
Nói như vậy cũng không có nghĩa rằng bạn không thể lập trình đa luồng với Rx. Rx cung cấp cho chúng ta 1 cơ chế xử lý đa luồng rất tiện dụng và hữu ích, đó chính là Scheduling.

*Scheduler:

Về cơ bản thì 1 Scheduler sẽ định nghĩa ra thread để chạy 1 khối lượng công việc. RxJava cung cấp những lựa chọn Scheduler như sau:

   immediate(): Tạo ra và trả về 1 Scheduler để thực thi công việc trên thread hiện tại.
   
   trampoline(): Tạo ra và trả về 1 Scheduler để sắp xếp 1 hàng chờ cho công việc trên thread hiện tại để thực thi khi công việc hiện tại kết thúc.
   
   newThread(): Tạo ra và trả về 1 Scheduler để tạo ra 1 thread mới cho mỗi đơn vị công việc.
   
   computation(): Tạo ra và trả về 1 Scheduler với mục đích xử lý các công việc tính toán, được hỗ trợ bởi 1 thread pool giới hạn với size bằng với số CPU hiện có.
   
   io(): Tạo ra và trả về 1 Scheduler với mục đích xử lý các công việc không mang nặng tính chất tính toán, được hỗ trợ bởi 1 thread pool không giới hạn có thể mở rộng khi cần.          Có thể được dùng để thực thi các tiến trình bất đồng bộ không gây ảnh hưởng lớn tới CPU.
   
   
Vị trí gọi subscribeOn() không quan trọng

Bạn có thể gọi hàm này ở bất cứ chỗ nào giữa Observable và Subscriber bởi vì nó chỉ có tác dụng khi hàm subscribe() được gọi đến
Bạn cần lưu ý điều này khi sử dụng các hàm như Observable.just(), Observable.from() hay Observable.range(): Những hàm này sẽ nhận vào giá trị ngay khi chúng được khởi tạo nên subscribeOn() sẽ không có tác dụng; Nguyên nhân là do subscribeOn() chỉ có tác dụng khi hàm subscribe() được gọi đến, mà những hàm khởi tạo nói trên lại khởi tạo Observable trước khi gọi subscriber() nên các bạn cần tránh đưa vào các giá trị mà cần tính toán trong 1 khoảng thời gian dài (blocking) vào các hàm khởi tạo đó.

Thay vào đó đối với các hàm blocking thì bạn có thể sử dụng Observable.create() hoặc Observable.defer(). 2 hàm này về cơ bản sẽ đảm bảo là Observable sẽ chỉ được khởi tạo khi hàm subscribe() được gọi đến.

Nếu bạn gọi nhiều lần hàm subscribeOn() với các Scheduler khác nhau thì cũng chỉ có hàm gọi đầu tiên từ trên xuống là có tác dụng thôi.

Hàm observeOn() nhận vào tham số là 1 Scheduler sẽ làm cho các Operator hay Subscriber được gọi đằng sau nó chạy trên thread được cung cấp bởi Scheduler đó.

