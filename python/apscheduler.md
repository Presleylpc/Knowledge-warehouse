> 官方文档[https://apscheduler.readthedocs.io/en/v3.5.1/userguide.html](https://apscheduler.readthedocs.io/en/v3.5.1/userguide.html)  
> 
> ### 安装

```
pip install apscheduler
```

### 调度选择

- **BlockingScheduler:** use when the scheduler is the only thing running in your process  
- **BackgroundScheduler:** use when you’re not using any of the frameworks below, and want the scheduler to run in the background inside your application  
- **AsyncIOScheduler:** use if your application uses the asyncio module  
- **GeventScheduler:** use if your application uses gevent  
- **TornadoScheduler:** use if you’re building a Tornado application  
- **TwistedScheduler:** use if you’re building a Twisted application  
- **QtScheduler:** use if you’re building a Qt application

### 触发器选择

- **date:** use when you want to run the job just once at a certain point of time  
- **interval:** use when you want to run the job at fixed intervals of time  

  ```
  job = scheduler.add_job(myfunc, 'interval', minutes=2)  
  job.remove()
  ```

  ```
  scheduler.add_job(myfunc, 'interval', minutes=2, id='my_job_id')  
  scheduler.remove_job('my_job_id')
  ```
- **cron:** use when you want to run the job periodically at certain time(s) of day

## 定义仓库

```
from pytz import utc

from apscheduler.schedulers.background import BackgroundScheduler  
from apscheduler.jobstores.mongodb import MongoDBJobStore  
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore  
from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor


jobstores = {  
'mongo': MongoDBJobStore(),  
'default': SQLAlchemyJobStore(url='sqlite:///jobs.sqlite')  
}  
executors = {  
'default': ThreadPoolExecutor(20),  
'processpool': ProcessPoolExecutor(5)  
}  
job_defaults = {  
'coalesce': False,  
'max_instances': 3  
}  
scheduler = BackgroundScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults, timezone=utc)
```

### 启动计划

```
start(paused=False)  
Start the configured executors and job stores and begin processing scheduled jobs.

Parameters: paused (bool) – if True, don’t start job processing until resume() is called  
Raises: SchedulerAlreadyRunningError – if the scheduler is already running
```

### 添加执行计划任务

> add_job(func, trigger=None, args=None, kwargs=None, id=None, name=None, misfire_grace_time=undefined, coalesce=undefined, max_instances=undefined, next_run_time=undefined, jobstore='default', executor='default', replace_existing=False, **trigger_args)  
> Adds the given job to the job list and wakes up the scheduler if it’s already running.

Any option that defaults to undefined will be replaced with the corresponding default value when the job is scheduled (which happens when the scheduler is started, or immediately if the scheduler is already running).

The func argument can be given either as a callable object or a textual reference in the package.module:some.object format, where the first half (separated by :) is an importable module and the second half is a reference to the callable object, relative to the module.

The trigger argument can either be:  

> the alias name of the trigger (e.g. date, interval or cron), in which case any extra keyword arguments to this method are passed on to the trigger’s constructor an instance of a trigger class

Parameters:  

- **func** – callable (or a textual reference to one) to run at the given time  
- **trigger** (str|apscheduler.triggers.base.BaseTrigger) – trigger that determines when func is called  
- **args** (list|tuple) – list of positional arguments to call func with  
- **kwargs** (dict) – dict of keyword arguments to call func with  
- **id** (str|unicode) – explicit identifier for the job (for modifying it later)  
- **name** (str|unicode) – textual description of the job  
- **misfire_grace_time** (int) – seconds after the designated runtime that the job is still allowed to be run  
- **coalesce** (bool) – run once instead of many times if the scheduler determines that the job should be run more than once in succession  
- **max_instances** (int) – maximum number of concurrently running instances allowed for this job  
- **next_run_time** (datetime) – when to first run the job, regardless of the - **trigger** (pass None to add the job as paused)  
- **jobstore** (str|unicode) – alias of the job store to store the job in  
- **executor** (str|unicode) – alias of the executor to run the job with  
- **replace_existing** (bool) – True to replace an existing job with the same - **id** (but retain the number of runs from the existing one)

Return type:  
**Job**

### 暂停/恢复Job

- 暂停 

```  
pause_job(job_id, jobstore=None)  
Causes the given job not to be executed until it is explicitly resumed.

Parameters:  
job_id (str|unicode) – the identifier of the job  
jobstore (str|unicode) – alias of the job store that contains the job  
Return Job:  
the relevant job instance  

```
- 恢复
```

resume_job(job_id, jobstore=None)  
Resumes the schedule of the given job, or removes the job if its schedule is finished.

Parameters:  
job_id (str|unicode) – the identifier of the job  
jobstore (str|unicode) – alias of the job store that contains the job  
Return Job|None:  

the relevant job instance if the job was rescheduled, or None if no next run time could be calculated and the job was removed  

```
### 实例
```
schedulers.py

from threading import Thread  
from datetime import date  
from apscheduler.schedulers.blocking import BlockingScheduler  
from apscheduler.jobstores.memory import MemoryJobStore  
from urllib import request  
import pytz

# 计划任务执行的方法

def _get_url(sid, url):  
parameter_str = "?sid={sid}&version={version}".format(sid=sid, version=date.today())  
url = url+parameter_str  
req = request.Request(url)  
page = request.urlopen(req).read()  
page = page.decode('utf-8')  

#print(page)

class _Scheduler:  
def __init__(self):  
self.job_store = MemoryJobStore() # 创建内存JobStore  
jobs_tores = {  
'default': self.job_store, # 定义JobStore  
}  
tz = pytz.timezone('Asia/Shanghai') # 定义时区为上海时区  
self._scheduler = BlockingScheduler(jobstores=jobs_tores, time_zone=tz) # 用后台模式创建一个执行计划，使用时区为上海时区  
thread = Thread(target=self._scheduler.start) # 多线程中执行计划任务  
thread.start()

def add_job_url(self, sid, url, cron_str):  
minute, hour, day, month, day_of_week = cron_str.split()  
self._scheduler.add_job(_get_url, 'cron', args=(sid, url), minute=minute,hour=hour, day=day, month=month, day_of_week=day_of_week,id=str(sid), name=str(sid), replace_existing=True) # 向计划中添加任务

def remove_job(self, sid):  
self._scheduler.remove_job(str(sid)) # 删除任务

def get_job(self, sid):  
return self._scheduler.get_job(job_id=sid) # 查看任务信息

def get_jobs(self):  
return self._scheduler.get_job() #查看所有任务信息

Scheduler = _Scheduler()  

```
```  
Add_listener.py

from slave_server.schedulers import Scheduler  
from time import sleep  
from apscheduler import events  
program_sid = 1  
program_cron_str_1 = '* * * * *'  
program_cron_str_2 = '*/2 * * * *'


def my_listener(event):  
print(event.job_id)  
print(event.scheduled_run_times[0].strftime("%Y-%m-%d %H:%M:%S"))  
print(Scheduler.get_job(event.job_id).next_run_time.strftime("%Y-%m-%d %H:%M:%S"))  
print(Scheduler.get_job(event.job_id).trigger)


Scheduler.add_job_url(sid=1, url="[http://www.baidu.com](http://www.baidu.com/)", cron_str=program_cron_str_1)  
# Scheduler.add_job_url(sid=2, url="[http://www.baidu.com](http://www.baidu.com/)", cron_str=program_cron_str_2)  
sleep(1)

Scheduler._scheduler.add_listener(my_listener, events.EVENT_JOB_SUBMITTED)
```
