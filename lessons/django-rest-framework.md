# Learn django-rest-framework

เรามี Model และเราสามารถที่จะ Interact กับ Model และข้อมูลใน Application ของเราได้แล้ว สิ่งที่เราจะทำต่อไปก็คือ การสร้าง API เพื่อที่จะให้หน้าเว็บ (Client) มาเรียกใช้งานผ่าน REST เราจะใช้เครื่องมือเข้ามาช่วยคือ django-rest-framework

1. Install django-rest-framework ก่อนด้วยคำสั่ง `pip install djangorestframework`

2. ใส่ `rest_framework` ลงไปใน INSTALLED_APPS

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
]
```

3. ต่อไปเราจะทำการ สร้าง Serializer class กัน ก่อนอื่นทำความเข้าใจเรื่องของ Serialization กันก่อน

Serialization เป็นกระบวนการในการเปลี่ยน data structure หรือ object state ให้ไปอยู่ใน format ที่สามารถเก็บลง Database, Memory หรือ File เพื่อที่จะเอาไปใช้งานต่อได้ในรูปแบบของ JSON, XML, ContentTypes อื่นๆ

สร้างไฟล์ `polls/serializers.py`

```
from rest_framework import serializers
from .model import Question, Choice

class QuestionSerializer(serializers.ModelSerializer):
  class Meta:
    model = Question
    fields = ['id', 'question_text', 'published_date']
```

```
class ChoiceSerializer(serializers.ModelSerializer):
  class Meta:
    model = Choice
    fields = ['id', 'question', 'choice_text']
```

4. เมื่อเราสร้าง Serializer เสร็จแล้ว เพื่อให้เห็นภาพเรามาลองเล่น Serializer กันซักหน่อยว่ามันนำไปใช้งานเบื้องต้นยังไง เราจะทำการสร้าง API ในการจัดการ Polls

เราจะทำการสร้าง API สำหรับ Questions ในการ list questions ทั้งหมด และสร้าง question ที่ `polls/views.py`

```
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from polls.models import Question
from polls.serializers import QuestionSerializer

@api_view(['GET', 'POST'])
def question_list(request):
  if request.method == 'GET':
    questions = Question.objects.all()
    serializer = QuestionSerializer(questions, many=True)
    return Response({ data: serializer.data })

  if request.method == 'POST'
    serializer = QuestionSerializer(data=request.data)
    if serializer.is_valid():
      serializer.save()
      return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.error, status=status.HTTP_400_BAD_REQUEST)
```

5. สร้าง API สำหรับจัดการ Question รายตัว

```
@api_view(['GET', 'POST', 'DELETE'])
def question_detail(request, question_id):
  try:
    question = Snippet.objects.get(pk=question_id)
  except Question.DoesNotExist:
    return Response(status=status.HTTP_404_NOT_FOUND)

  if request.method == 'GET':
    serializer = QuestionSerializer(question)
    return Response(serializer.data)

  elif request.method == 'PUT':
    serializer = QuestionSerializer(question, request.data)
    if serializer.is_valid():
      serializer.save()
      return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

  elif request.method == 'DELETE':
    question.delete()
    return Response(status=status.HTTP_204_NO_CONTENT)
```

6. เมื่อเราสร้าง API เสร็จเรียบร้อยแล้ว ให้เราไป Update URL ที่ `polls/urls.py`

```
from django.urls import path
from polls import views

urlpatterns = [
  path('questions/', views.question_list),
  path('questions/<int:question_id>, views.question_detail)
]
```
