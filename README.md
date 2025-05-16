# Accuknox_Assignment


Question 1: By default are django signals executed synchronously or asynchronously? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Answer:---

Django signals are synchronous by default. This means that when a signal is triggered all connected functions are executed one after another and the code that triggered the signal waits for all of them to complete before proceeding.

--> First need to create signal.py file inside app folder.

from django.dispatch import signal 
from django.db.models.signals import post_save
from django.contrib.auth.models import User
import time

My_signal = signal()
def First_receiver(sender,**param):
    print("Receiver First Started")
    time.sleep(2)     #-------> delay for 2 sec
    print("Receiver First Finished")
def Second_receiver(sender, **param):
    print("Receiver Second Executed Immediately After Receiver First")

my_signal.connect(receiver_one)
my_signal.connect(receiver_two)


---> Now import signal in views.py and creare views.

from django.shortcuts import HttpResponse
from .signals import my_signal

def trigger_signal(request):
    print("Signal sent.....")
    Mysignal.send(sender=trigger_signal)
    print("Signal sending finished.")
    return HttpResponse("Signal triggered!")

points:-----

--> define a custom signal Mysignal.
--> create two receiver functions: First_Receiver which pauses for 2 second and ----> Second_Receiver which prints a message.
--> connect both Second_Receive and Second_Receiver to Mysignal.
--> In the trigger_signal view.we send the Mysignal.


----------------------------------------------------------------------------------

Question 2: Do django signals run in the same thread as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Answer:---
sihgnal are by default runs in the same thread as the code that triggred them.
Both the main code and the signal function run together in the same flow.


-->  First need to create signal.py file inside app folder.


from django.dispatch import Signal
import threading

thread_signal = Signal()

def signal_receiver(sender, **params):
   print("Signal received in thread: {}".format(threading.current_thread().name))

thread_signal.connect(signal_receiver)


---> Now import signal in views.py and creare views.


from django.shortcuts import HttpResponse
from .signals import thread_signal
import threading

def send_thread_signal(request):
   print("Signal sent for thread: {}".format(threading.current_thread().name))
    thread_signal.send(sender=send_thread_signal)
    return HttpResponse("Thread signal sent!")


--> Define a signal thread_signal.
--> The signal_receiver function uses the threading.current_thread().name to get and print the name of the thread in which it's running.
--> connect signal_receiver to thread_signal.
--> The send_thread_signal view gets the name of the current thread before sending the thread_signal.



----------------------------------------------------------------------------------


Question 3: By default do django signals run in the same database transaction as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Answer--> The django signals are by deafult don not run in the same database transcation as the code that sends the siganl.

--> When a signal is triggered the connected receiver functions run afterward. If a receiver makes changes to the database those changes happen in a separate transaction not in the same transaction as the code that triggered the signal.


--> creates models.py

from django.db import models

class EventLog(models.Model):
    description = models.TextField()

class MyModel(models.Model):
    name = models.CharField(max_length=100)

--> creates signal.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.db import transaction
from .models import MyModel, EventLog

@receiver(post_save, sender=MyModel)
def log_my_model_creation(sender, instance, created, **kwargs):
    try:
        with transaction.atomic():
            EventLog.objects.create(description=f"MyModel '{instance.name}' was created.")
           transaction
            if instance.name == "Trigger Error":
                raise ValueError("Simulating error in signal receiver")
    except ValueError as e:
        print("Error in signal receiver: {}".format(e))

--> creates views.py

from django.shortcuts import HttpResponse
from django.db import transaction
from .models import MyModel, EventLog

def create_my_model(request):
    try:
        with transaction.atomic():
            obj = MyModel.objects.create(name="Test Object")
            return HttpResponse(f"MyModel created with ID: {obj.id}")
    except Exception as e:
        return HttpResponse(f"Error creating MyModel: {e}", status=500)

def create_my_model_trigger_error(request):
    try:
        with transaction.atomic():
            obj = MyModel.objects.create(name="Trigger Error")
            return HttpResponse(f"MyModel created with ID: {obj.id} (will trigger receiver error)")
    except Exception as e:
        return HttpResponse(f"Error creating MyModel: {e}", status=500)

def view_event_logs(request):
    logs = EventLog.objects.all()
    log_entries = "<br>".join([log.description for log in logs])
    return HttpResponse(f"Event Logs:<br>{log_entries}")




    
