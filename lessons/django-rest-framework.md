# Learn django-rest-framework

เรามี Model และเราสามารถที่จะ Interact กับ Model และข้อมูลใน Application ของเราได้แล้ว สิ่งที่เราจะทำต่อไปก็คือ การสร้าง API เพื่อที่จะให้หน้าเว็บ (Client) มาเรียกใช้งานผ่าน REST เราจะใช้เครื่องมือเข้ามาช่วยคือ django-rest-framework

1. Install django-rest-framework ก่อนด้วยคำสั่ง `pip install djangorestframework`

2. ใส่ `rest_framework` ลงไปใน INSTALLED_APPS

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
]
```

3. ต่อไปเราจะทำการ สร้าง Serializer class กัน ก่อนอื่นทำความเข้าใจเรื่องของ Serialization กันก่อน

Serialization เป็นกระบวนการในการเปลี่ยน data structure หรือ object state ให้ไปอยู่ใน format ที่สามารถเก็บลง Database, Memory หรือ File จุดประสงค์หลักคือ เป็นการบันทึกข้อมูล หรือ state นั้น ๆ เอาไว้ เพื่อใช้ในการส่งผ่านข้อมูลไปยังอีกระบบ หรือบันทึกลง database

สร้างไฟล์ `polls/serializers.py`

```python
from rest_framework import serializers
from .model import Question, Choice

class QuestionSerializer(serializers.ModelSerializer):
  class Meta:
    model = Question
    fields = ['id', 'question_text', 'published_date']
```

```python
class ChoiceSerializer(serializers.ModelSerializer):
  class Meta:
    model = Choice
    fields = ['id', 'question', 'choice_text']
```

4. เมื่อเราสร้าง Serializer เสร็จแล้ว เพื่อให้เห็นภาพเรามาลองเล่น Serializer กันซักหน่อยว่ามันนำไปใช้งานเบื้องต้นยังไง เราจะทำการสร้าง API ในการจัดการ Polls

เราจะทำการสร้าง API สำหรับ Questions ในการ list questions ทั้งหมด ที่ `polls/views.py`

```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from polls.models import Question
from polls.serializers import QuestionSerializer

@api_view(['GET'])
def question_list(request):
  if request.method == 'GET':
    questions = Question.objects.all()
    serializer = QuestionSerializer(questions, many=True)
    return Response({ "data": serializer.data })
```

5. เราจะทำการลองเล่น API Get Questions ทั้งหมด ด้วย Postman เราจะเห็นผลลัพธ์คล้ายๆตัวอย่างข้างล่างนี้

```json
{
    "data": [
        {
            "id": 10,
            "question_text": "question_text test",
            "pub_date": "2020-12-19T02:31:38Z"
        }
    ]
}
```

6. ต่อไปเราจะทำการสร้าง API สำหรับการสร้าง Question เราจะเพิ่มโค้ดใน function question list ของเราต่อ เราจะเช็ค `request.method` เพิ่มว่า ถ้า request ที่เข้ามาผ่าน url `polls/questions/` และเป็น POST ให้ทำการสร้าง Question นั้น

```python
@api_view(['GET', 'POST'])
def question_list(request):
  if request.method == 'GET':
    questions = Question.objects.all()
    serializer = QuestionSerializer(questions, many=True)
    return Response({ "data": serializer.data })

  if request.method == 'POST':
    serializer = QuestionSerializer(data=request.data)
    if serializer.is_valid():
      serializer.save()
      return Response({ "message": "create successfully", "data": serializer.data }, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

6. เราจะทำการลองใช้ Postman ในการลองเล่นกับ API Create Question ที่เราสร้าง

ปรับ HTTP Method เป็น POST และใส่ Request body ดังนี้

```json
{
    "question_text": "question_text test",
    "pub_date": "2020-12-19T02:31:38Z"
}
```

เราจะได้ Response กลับมาดังนี้

```json
{
    "message": "create successfully",
    "data": {
        "id": 13,
        "question_text": "question_text test",
        "pub_date": "2020-12-19T02:31:38Z"
    }
}
```

7. เนื่องจากว่า Question ของเราสามารถมี Choices ได้หลายตัว เราจะทำการปรับ Code นิดหน่อยให้เราสามารถสร้าง Choices พร้อม ๆ กับตัว Question ไปด้วยเลยใน API เดียวกัน เราจะทำการการปรับ serializers ที่ `polls/serializers.py` ด้วยการเพิ่ม function create เข้าไปใน `QuestionSerializer` Class ดังนี้

```python
class QuestionSerializer(serializers.ModelSerializer):
  choices = ChoiceSerializer(many=True)

  class Meta:
    model = Question
    fields = ['id', 'question_text', 'pub_date', 'choices']

  def create(self, validated_data):
    choices_data = validated_data.pop('choices')
    question = Question.objects.create(**validated_data)
    for choice_data in choices_data:
        Choice.objects.create(question=question, **choice_data)
    return question
```

8. เราจะทำการสร้าง Question และ Choices ผ่าน Postman

ทำการใส่ Request body ดังนี้

```json
{
    "question_text": "question_text test",
    "pub_date": "2020-12-19T02:31:38Z",
    "choices": [
        {
            "choice_text": "choice a",
            "votes": 0
        },
        {
            "choice_text": "choice b",
            "votes": 5
        }
    ]
}
```

ผลลัพทธ์ที่ออกมาจะได้ประมาณนี้

```json
{
    "message": "create successfully",
    "data": {
        "id": 14,
        "question_text": "question_text test",
        "pub_date": "2020-12-19T02:31:38Z",
        "choices": [
            {
                "id": 9,
                "choice_text": "choice a",
                "votes": 0
            },
            {
                "id": 10,
                "choice_text": "choice b",
                "votes": 5
            }
        ]
    }
}
```

9. เราจะสร้าง API สำหรับการจัดการข้อมูลของ Question รายตัว ได้แก่ การ Read, Update, Delete ต่อมาให้เราไป Update URL ที่ `polls/urls.py` ตามนี้ ด้วยการเพิ่ม path ใหม่เข้าไป ซึ่ง path นี้จะทำการรับ question id ที่เป็น Integer แล้วเราจะทำการสร้าง views ใหม่ที่ชื่อว่า `question_detail`

```python
from django.urls import path
from polls import views

urlpatterns = [
  path('questions/', views.question_list),
  path('questions/<int:question_id>, views.question_detail)
]
```

10. ให้เรามาที่ `polls/views.py` เราจะสร้าง Function ใหม่ ชืื่อว่า `question_detail` แล้วเราจะทำการดึงข้อมูลของ Question นั้น ๆ ด้วย question_id ที่ส่งมาจาก url ที่เราเขียน path ไว้รับ แล้วเราจะเขียนเพิ่มว่า ถ้า request ที่เข้ามาเป็น GET เราจะให้มัน return ข้อมูลของ Question นั้น ออกไปได้เลย ตามโค้ดด้านล่าง

```python
def question_detail(request, question_id):
  try:
    question = Question.objects.get(pk=question_id)
  except Question.DoesNotExist:
    return Response({ "message": "requested question not found" }, status=status.HTTP_404_NOT_FOUND)

  if request.method == 'GET':
    serializer = QuestionSerializer(question)
    return Response({ "data": serializer.data })
```

11. เราจะลองดึงข้อมูลของ Question ที่เราสร้างไปเมื่อสักครู่นี้ผ่าน Postman

ทำการเปลี่ยน Method เป็น GET และ Url เป็น `localhost:8000/polls/question/14`

*ตัวเลข 14 เป็น Id ของ Question ที่อยากจะดึงอาจจะไม่เหมือนกันทุกเครื่อง ให้อิงตามเลขที่สร้างจากเครื่องตัวเอง*

เราจะได้ Response มาหน้าตาประมาณนี้

```
{
    "data": {
        "id": 14,
        "question_text": "question_text test",
        "pub_date": "2020-12-19T02:31:38Z",
        "choices": [
            {
                "id": 9,
                "choice_text": "choice a",
                "votes": 0
            },
            {
                "id": 10,
                "choice_text": "choice b",
                "votes": 5
            }
        ]
    }
}
```

12. จากนั้นเราจะทำการสร้าง API สำหรับการลบ Question เราจะทำการเช็คเงื่อนไขเพิ่ม ถ้า `request.method` เป็น `DELETE` ให้ทำการลบตัว Question ออกจาก Database

```python
...

  if request.method == 'DELETE':
      serializer = QuestionSerializer(question)
      question_id = serializer.data['id']
      test = question.delete()
      return Response({"message": f"question {question_id} delete successfully"}, status=status.HTTP_204_NO_CONTENT)
```

13. ต่อมาเราจะทำการ Update ข้อมูลของ Question และ Choices ใน API เดียวกันตรงนี้อาจจะวุ่นวายสักนิดหน่อย แต่ลองดูนะครับถ้าผ่านตรงนี้ได้ จะทำให้เราสามารถเขียน API ในการจัดการข้อมูลได้เก่งมากยิ่งขึ้น

ก่อนอื่นเลยเหมือนเดิม เราจะทำการเพิ่มเงื่อนไข ถ้า `request.method` เป็น `PUT` ให้ทำการ Update ข้อมูล

```python
...

  if request.method == 'PUT':
    serializer = QuestionSerializer(question, data=request.data)
    if serializer.is_valid():
      serializer.save()
      return Response({ "message": "update successfully", "data": serializer.data })
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

14. ทำการแก้ไข `polls/serializers.py` เพื่อให้เราสามารถ Update Question และ Choices ได้ใน API เดียวกัน ด้วยการเพิ่ม Function update เข้าไปข้างใน `QuestionSerializer` Class

```python
def update(self, instance, validated_data):
    instance.question_text = validated_data.get(
      'question_text', instance.question_text)
    instance.pub_date = validated_data.get(
        'pub_date', instance.pub_date)
    instance.save()

    input_choices = validated_data.pop('choices')
    choices = (instance.choices).all()
    choices = list(choices)
    for input_choice in input_choices:
        if not len(choices):
          Choice.objects.create(question=instance, **input_choice)
          continue
        choice = choices.pop(0)
        choice.choice_text = input_choice.get(
            'choice_text', choice.choice_text)
        choice.votes = input_choice.get('votes', choice.votes)
        choice.save()

    return instance
```

15. จากนั้นลองทำการ Update ผ่าน Postman

ใส่ Request Body ดังนี้

```json
{
    "question_text": "new question_text test2",
    "pub_date": "2020-12-19T02:31:38Z",
    "choices": [
        {
            "id": 5,
            "choice_text": "updated choice",
            "votes": 10
        },
        {
            "choice_text": "update new choice 222",
            "votes": 20
        },
        {
             "choice_text": "update new choice 30",
            "votes": 30
        }
    ]
}
```

หน้าตา Reponse จะเป็นประมาณนี้

```json
{
    "message": "update successfully",
    "data": {
        "id": 12,
        "question_text": "new question_text test2",
        "pub_date": "2020-12-19T02:31:38Z",
        "choices": [
            {
                "id": 5,
                "choice_text": "updated choice",
                "votes": 10
            },
            {
                "id": 6,
                "choice_text": "update new choice 222",
                "votes": 20
            },
            {
                "id": 8,
                "choice_text": "update new choice 30",
                "votes": 30
            }
        ]
    }
}
```

*ถ้าเราใส่ Chioce ที่ไม่ได้มีอยู่ใน Database ตัว API เราจะทำการสร้างให้อัตโนมัติตามโค้ดที่เราเขียนกันไว้จำได้มั้ยนะ*