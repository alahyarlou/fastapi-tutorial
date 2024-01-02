# Path Parameters

خب رسیدیم به قسمتی که ما میتونیم به همراه url که میفرستیم بتونیم پارامتر یا متغییر هایی را ارسال کنیم.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

قبل از توضیحات کد بالا شما در نظر بگیرید که داخل یه سایتی شدید که به این صورت است:

```bash
https://github.com/alahyarlou
```

خب در اینجا اگه دقت کنید تا این قسمت `https://github.com` آدرس دامین سایت گیتهاب رو اشاره میکنه.
و اگه به پارامتر بعدیش دقت کنید میبنید که یه `user_id` هستش که وقتی بعد از آدرس سرور وارد میکنیم ما رو مستقیما به صفحه اون کاربر میبره!

---

حالا تو کد بالا میبنید که برای خوندن خبر خاص بعدش باید پارامتر یا آیدی اون خبر رو وارد کنیم که بتونیم به اون خبر اگه وجود داشته باشه برسیم.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
   return {"item_id": item_id}
```

شما همچنین میتونید همزمان که دارید پارامتری که میفرستید نوع اون رو نیز مشخص کنید.

واگر نوعی که وارد شده رو اشتباهی string وارد کنید بهتون خطا برمیگردونه!

```bash
http://127.0.0.1:8000/items/foo
```

Return:

```json
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": ["path", "item_id"],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "foo",
      "url": "https://errors.pydantic.dev/2.1/v/int_parsing"
    }
  ]
}
```

خطایی که بهش اشاره میکنه اگه دقت کنید کاملا واضح هست که میگه پارامتری که وارد کردید باید نوع اش int باشه ولی شما string وارد کرده اید.

## Pydantic

تمامی اعتبار سنجی های داده ها توسط pydantic انجام میشود.
یه چندتا از نوع داده هایی که میتوانید استفاده کنید:

- int
- str
- float
- bool

و تعداد زیاد دیگه ای رو قراره تو آینده بهشون اشاره کنیم!

---

ممکنه براتون یه مشکلی هنگام استفاده از path parameter ها پیش بیاد!
خب به نظرتون ممکنه اون مشکل چی باشه؟؟🤔

اول یه نگاهی به کد زیر بندازید:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}
```

خب بعدش به آدرس زیر درخواست بدین تا متوجه مشکل بشین :))

```bash
http://127.0.0.1:8000/users/me
```

و چیزی که براتون برمیگردونه به این صورت است:

```json
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": ["path", "user_id"],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "me",
      "url": "https://errors.pydantic.dev/2.1/v/int_parsing"
    }
  ]
}
```

با خودتون شاید فکر کنید که چرا این مشکل به وجود اومده با اینکه من روت های این آدرس رو بهمراه پارامترش براش مشخص کردم ولی در اینجا باید دقت کنید که قرار گیری تابع ها خیلی مهم هستند و وقتی کد از بالا خونده میشه اولین چیزی که درنظر میگیره روت بالایی هستش و برای حل کردن این مشکل باید جای تابع ها رو باهمدیگه عوض کنیم.

نتیجه:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

## Predefined values

تو این قسمت میخواهیم به مقادیر از پیش تعریف شده اشاره کنیم.

شما فرض کنید یه route دارید که پارامتر هایی که میخواهید دریافت کنید باید خاص باشند یا مورد قبول شما باشند برای اینکار باید اون مقادیر رو از پیش براش تعریف کنید که اگه غیر از این مقادیر بود براش خطا چاپ کنه!

خب برای اینکار باید از enum ها استفاده کنیم.

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```