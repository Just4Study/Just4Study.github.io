---
layout: post
title: DRF(Django Rest Framework) Tutorial (1)
categories: ['Web-Backend']
excerpt: Django Rest Framework 기초 쌓기 1
tags: ['Django Rest Framework']
toc: true
---

### Django Rest Framework 공식 문서 [Offical Site](https://www.django-rest-framework.org/)

- 개발 환경 : Ubuntu 20.04.2
- IDE : VSCode

0. Coding Style
    - 함수는 Cammel Case로
    - 되도록 간결하게

1. Python 가상환경 설정 (Venv)
    - sudo apt-get update
    - sudo apt-get python3-venv
    - python3 -m venv venv

2. Python 가상환경 켜기
    - source venv/bin/activate

3. DRF 설치
    - pip install djangorestframework (Django가 없을 경우 알아서 설치)

4. 프로젝트 설치
    - djando-admin startproject todo .
    - settings.py INSTALLED_APPS에 rest_framework 추가
    - rest api인 만큼 api만 관리하는 application 설정
        - python manage.py startapp api
        - settings.py INSTALLED_APPS에 api 추가

5. 기본설정
    - model 설정
    - urls 설정
    - view 설정

6. 실행
    - python manage.py migrate
    - python manage.py runserver
    - 테스트 127.0.0.1:8000/api/ 접속 시 "API BASE POINT" 보임

7. DRF Requests and Responses [offical docs](https://www.django-rest-framework.org/tutorial/2-requests-and-responses/)
    - CBV, FBV 중 FBV 선택
    - from rest_framework.decorators import api_view from rest_framework.response import Response 추가
    - FBV 일 시 @api_view decorator 사용
        - @api_view(['GET', 'POST'])로 메소드 지정 (GET, PUT, DELETE, POST 있음)
    - DRF View에서 api_url 설정 가능

        ```python
        # api/views.py
        @api_view(['GET'])
        def apiOverview(request):
            api_urls = {
                'List': '/task-list/',
                'Detail View': '/task-detail/<str:pk>/',
                'Create': '/task-create/',
                'Update': '/task-update/<str:pk>/',
                'Delete': '/task-delete/<str:pk>'
            }
            return Response(api_urls)
        }
        ```

        ```python
        # api/urls.py
        from django.urls import path
        from . import views

        urlpatterns = [
            path('', views.apiOverview, name="api-overview"),
        ]
        ```

8. Serializers
    - App Folder에서 serializers.py 생성

        ```python
        # api/serializers.py
        from rest_framework import serializers 추가!
        from .models import Task

        class TaskSerializer(serializers.ModelSerializer):
            class Meta:
                model = Task
                fields = '__all__' # every field
        ```

    - View에 serializer 추가

        ```python
        # api/views.py
        from .serializers import TaskSerializer # Django에서 Form 역할
        ```

9. taskList View:
    1. views.py 작성

        ```python
        # api/views.py
        @api_view(['GET'])
        def taskList(request):
            tasks = Task.objects.all()
            serializer = TaskSerializer(tasks, many=True) # More than One Objects many=True
            return Response(serializer.data)
        ```

    2. urls.py 작성

        ```python
        # api/urls.py
        urlpatterns = [
            ### URLS ###
            path('task-list/', views.taskList, name="api-list"), # Added URL!
        ]
        ```

10. taskDetail View:
    1. views.py 작성

        ```python
        # api/views.py
        @api_view(['GET'])
        def taskDetail(request, pk):
            tasks = Task.objects.get(id=pk)
            serializer = TaskSerializer(tasks, many=False) # Just One Object many=False
            return Response(serializer.data)
        ```

    2. urls.py 작성

        ```python
        # api/urls.py
        urlpatterns = [
            ### URLS ###
            path('task-detail/<str:pk>/', views.taskDetail, name="task-detail"), # Added URL!
        ]
        ```

11. taskCreate View:
    1. views.py 작성

        request.POST 대신 request.data 쓰는 이유
        
        ![image](https://user-images.githubusercontent.com/48237469/126114431-d0f87df5-d1fd-41df-a18d-4089c7fef325.png)

        ```python
        # api/views.py
        @api_view(['POST'])
        def taskCreate(request):
            serializer = TaskSerializer(data=request.data) # API View, request.data Not request.POOST
            
            if serializer.is_valid():
                serializer.save()

            return Response(serializer.data)
        ```

    2. urls.py 작성

        ```python
        # api/urls.py
        urlpatterns = [
            ### URLS ###
            path('task-create/', views.taskCreate, name="task-create"), # Added URL!
        ]
        ```

    3. Example json

        ```json
        // 127:0.0.1:8000/api/task-create/
        {
           "title": "Todo APP Job"
        }
        ```

        ![image](https://user-images.githubusercontent.com/48237469/126114945-ba9e5050-ee22-4935-9018-d764c6fc5d67.png)
        <small>ID를 적지 않아도 알아서 순차적으로 들어간다.</small>

        ![image](https://user-images.githubusercontent.com/48237469/126115055-3ba1e4c2-60bf-4f86-9ca6-06ebce36b2cf.png)
        <small>Task List로 결과를 확인한 화면</small>
12. taskUpdate View
    1. views.py 작성

        ```python
        # api/views.py
        @api_view(['POST'])
        def taskUpdate(request, pk):
            task = Task.objects.get(id=pk) # GET Data
            serializer = TaskSerializer(instance=task, data=request.data) # instance means existing object
            
            if serializer.is_valid():
                serializer.save()

            return Response(serializer.data)
        ```

    2. urls.py 작성

        ```python
        # api/urls.py
        urlpatterns = [
            ### URLS ###
            path('task-update/<str:pk>/', views.taskUpdate, name="task-update"),
        ]
        ```

    3. Example json

        ```json
        // 127:0.0.1:8000/api/task-update/2
        {
           "title": "Todo APP Job(Updated!)"
        }
        ```

        ![image](https://user-images.githubusercontent.com/48237469/126115557-8839035c-6d5d-472a-81c3-7fb42459821c.png)

        ![image](https://user-images.githubusercontent.com/48237469/126115629-8a9441d1-6b07-4d38-896c-8f429b4bdd1a.png)
        - Post를 누르면 바로 결과를 확인할 수 있다.

13. taskDelete View
    1. views.py 작성

        ```python
        # api/views.py
        @api_view(['DELETE'])
        def taskDelete(request, pk):
            task = Task.objects.get(id=pk) # GET Data
            task.delete()

            return Response('Object successfully delete')
        ```

    2. urls.py 작성

        ```python
        # api/urls.py
        urlpatterns = [
            ### URLS ###
            path('task-delete/<str:pk>/', views.taskDelete, name="task-delete"),
        ]
        ```

    3. Example

        ![image](https://user-images.githubusercontent.com/48237469/126115975-289d95c2-56ba-42ae-958f-662ab96b5b45.png)

        ![image](https://user-images.githubusercontent.com/48237469/126116031-9f814735-1e84-4d8f-812b-200b704b2554.png)

### 마치며
- 다음에는 AJAX를 사용하여 DRF와 프론트엔드를 연결해보자!는 군대감 ㅜ