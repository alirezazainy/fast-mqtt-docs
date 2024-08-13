# **مستندات فنی**

## **کنترلر و ماژول ها**

### **`Unit of work`**
 
بخش کنترلر نرم افزار ما در واقع بر اساس دیزاین پترن `Unit of work` نوشته شده که در ساختار  `MVC` بسیار کارآمد بوده و نرم افزار را از خطرات ارتباط با دیتابیس مصون میکند.

در واقع این دیزاین پترن به ما شیوه نامه ای جهت بازنویسی `ORM` ها میدهد، که با توجه به آن لایه کنترلر نرم افزار ما به دو بخش ارتباط با دیتابیس و ارتباط با کد تقسیم می شود و دیگر نمیتوان مستقیما از طریق روتر(در اصطلاح اصلی ویو)ها یک در خواست به دیتابیس ارسال کرد. 

از آنجایی که خود ساختار `MVC` کنترلر را برای همین منظور ایجاد کرده بود؛ اما متاسفانه بدلیل اینکه می شد به صورت مستقیم یک کوئری به دیتابیس ارسال کرد، این دیزاین پترن طراحی شد تا با استفاده از آن از کوئری مستقیم به دیتابیس خودداری شود و ساختار را امن نگه دارد.

برای مثال در این نرم افزار: 

=== "کد فایل crud.py"
    ```{.py .no-copy}
    
    def create(db: Session, request: MessageBaseModel):
        message = Message(imei=request.imei, micro_op=request.micro_op)  # -> add instance
        # add instance to database
        db.add(message)
        db.commit()
        db.refresh(message)
        return True
    ```

=== "کد فایل test_controller.py"
    ```{.py .no-copy}
    def save_message(db: Session, request: MessageBaseModel):
        try:  # try connecting database and save data
            result = crud.create(db, request)
            if not result:  # handling error
                not_error = False
            return result
        except not_error:
            return not_error
    ```

### **پیاده سازی کلی Paho-MQTT**

طبق مستندات این کتابخانه صرفا به ایجاد توابع `connect_mqtt`, `on_connect` و `publish` بسنده کردیم.

در این ماژول یک متغیر گلوبال به نام `CLIENT` ایجاد کردیم که با استفاده از کنترلر که به ماژول متصل میشود، فراخوانی شده و از کلاینت MQTT نمونه سازی کند.

=== "کد فایل test_controller.py"
    ```{.py .no-copy}
    async def connect_mqtt_broker():
    try:
        result = CLIENT.loop_start()
        if not result:
            not_error = False
    except not_error:
        return not_error
    ```

=== "کد فایل mqtt_publisher.py"
    ```{.py .no-copy}
    def connect_mqtt():
        def on_connect(client, user_data, flags, response_code, properties):
            if response_code == 0:
                return "response_message": "Connected to MQTT Broker!"
            else:
                return f"Failed to connect, return code {response_code}"
        client = mqtt_client.Client(CallbackAPIVersion.VERSION2, client_id)
        # client.username_pw_set(username, password)
        client.on_connect = on_connect
        client.connect(broker_server, port)
        return client
    ```
متغیر `CLIENT` به صورت بالا از طریق کنترلر فراخوانی شده و به بروکر متصل می شود. همچنین میتوانید مشاهده کنید که پس از فراخوانی `CLIENT` چگونه به کلاینت متصل می شویم.
