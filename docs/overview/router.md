# **مستندات فنی**

## **روترها**

### **نحوه برقراری اتصال میان کلاینت و بروکر**

برای اتصال میان بروکر و کلاینت و ایجاد لوپ، از قابلیت startup event در FastAPI استفاده شده تا در هنگام راه اندازی برنامه یک لوپ به صورت `Async` ایجاد بشود.

=== "کد فایل application.py"
    ```{.py hl_lines="3" .no-copy}
    @app.on_event("startup")
    async def on_startup() -> None:
        await connect_mqtt_broker()
    ```
??? question "چرا `Async` ؟"
    این لوپ به این دلیل `Async` توسعه داده شد تا در هنگام اجرا اگر به مشکلی برخورد برای اتصال مجدد به بروکر نیاز به راه اندازی مجدد اپلیکیشن نباشد.

تابع `on_startup` که هنگام اجرا در حالت `Await` تابع `connect_mqtt_broker` را فراخوانی میکند این وظیفه مهم را بر عهده دارد.

### **چگونگی ارسال یک پیام**

در روتر `Messaging` یک `API` وجود دارد که از طریق فراخوانی مسیر `/messaging/send` ابتدا پیام مورد نظر را دریافت کرده و به تابع `send_message` در حالت `Await` می دهد تا این تابع برود و این پیام را به تابع فرستنده به کمک کنترلر برساند.

=== "کد فایل messaging.py" 
    ```{.py hl_lines="5" .no-copy}
    @router.post("/send")
    async def send_and_save_massage(
        request: MessageBaseModel, db: Session = Depends(get_db)
    ):
        result = await send_message(request)  # -> sending message
        if result is not str:
            # send request to database for saving
            data = save_message(db, request)
            while not data:
                data = save_message(db, request)
            if not data:
                raise HTTPException(status.HTTP_406_NOT_ACCEPTABLE, "Something went wrong")
            raise HTTPException(status.HTTP_201_CREATED, result)
        else:
            raise HTTPException(status.HTTP_406_NOT_ACCEPTABLE, "Something went wrong")
    ```
پس از اینکه این تابع توانست پیام را ارسال کند، نتیجه اش توسط یک شرط بررسی می شود تا بتوانیم آن پیام را در صورت موفقیت در دیتابیس ذخیره کنیم. 

به همین منظور پس از بررسی، داده را به تابع `save_message` می دهیم تا به کمک کنترلر برود و داده را در دیتابیس پروژه ذخیره سازی کند.

=== "کد فایل messaging.py" 
    ```{.py hl_lines="6-12" .no-copy}
    @router.post("/send")
    async def send_and_save_massage(
        request: MessageBaseModel, db: Session = Depends(get_db)
    ):
        result = await send_message(request)  # -> sending message
        if result is not str:
            # send request to database for saving
            data = save_message(db, request)
            while not data:
                data = save_message(db, request)
            if not data:
                raise HTTPException(status.HTTP_406_NOT_ACCEPTABLE, "Something went wrong")
            raise HTTPException(status.HTTP_201_CREATED, result)
        else:
            raise HTTPException(status.HTTP_406_NOT_ACCEPTABLE, "Something went wrong")
    ```

در صورت ارسال موفق داده و ذخیره سازی آن یک پیام با `status code` برابر با `201` بر میگرداند که نشانه موفقیت در انجام عملیات است؛ و در صورت شکست خوردن در هر یک از مراحل یک پیام با `status code` برابر با `406` که نشان دهنده پذیرفته نشدن داده توسط اپلیکیشن است.

=== "status code 200"
    ```json 
    {
        "Send `{published_message}` to topic `{topic}`"
    }
    ```
=== "status code 406"
    ```json 
    {
        "Somthing went wrong"
    }
    ```