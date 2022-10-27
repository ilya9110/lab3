# lab3
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
# 3 ЛАБОРАТОРНАЯ РАБОТА. РАЗРАБОТКА СИСТЕМЫ МАШИННОГО ОБУЧЕНИЯ.
Отчет по лабораторной работе #3 выполнил(а):
- Маслий Илья Андреевич
- РИ211102
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
Ход работы:
Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.
- Создайте новый пустой 3D проект на Unity.
![image](https://user-images.githubusercontent.com/29748577/198320308-476550d6-f9b4-42da-a5ef-7fb3d22a6ebb.png)

- В созданный проект добавьте ML Agent, выбрав Window - Package Manager - Add Package from disk.
Если все сделано правильно, то во вкладке с компонентами (Components) внутри Unity вы увидите строку ML Agent.
![image](https://user-images.githubusercontent.com/29748577/198321195-8607a148-2114-4112-9338-ddfbdf596a82.png)

- Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:
o mlagents 0.28.0;
o torch 1.7.1;
![image](https://user-images.githubusercontent.com/29748577/198321998-4ac4af91-2094-40bd-9189-b47d7b906787.png)
![image](https://user-images.githubusercontent.com/29748577/198322127-360ffc59-1199-42c3-87ed-475aa08a0cad.png)
![image](https://user-images.githubusercontent.com/29748577/198322286-666ea71e-af79-48ac-87b8-b9022c0579fe.png)
![image](https://user-images.githubusercontent.com/29748577/198322464-b6ebef79-3408-48e5-931b-09a54d14fd19.png)

- Создайте на сцене плоскость, куб и сферу так, как показано на рисунке ниже. Создайте простой C# скрипт-файл и подключите его к сфере:
![image](https://user-images.githubusercontent.com/29748577/198322863-ff3ddc15-a76e-4a3b-b072-e2066e28a23f.png)
- В скрипт-файл RollerAgent.cs добавьте код, опубликованный в материалах лабораторных работ – по ссылке.
Код скрипта:
```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```

- В корень проекта добавьте файл конфигурации нейронной сети,
доступный в папке с файлами проекта по ссылке.
Запустите работу ml-агента
![image](https://user-images.githubusercontent.com/29748577/198323717-058180e6-6e5a-473c-a400-47705250c98a.png)

-Вернитесь в проект Unity, запустите сцену, проверьте работу ML-Agent’a.
Сделайте 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запустите симуляцию сцены и наблюдайте за результатом обучения модели.
После завершения обучения проверьте работу модели.
![20221027_195707](https://user-images.githubusercontent.com/29748577/198325561-529d46a3-8ead-4054-9605-ebced8249cac.gif)

## Задание 2
### Подробно опишите каждую строку файла конфигурациинейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельнонайдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.
Ход работы:

```py

behaviors:
  RollerBall:                        #Имя агента
    trainer_type: ppo                #Устанавливаем режим обучения (Proximal Policy Optimization).
    hyperparameters:                 #Задаются гиперпараметры.
      batch_size: 10                 #Количество опытов на каждой итерации для обновления экстремумов функции.
      buffer_size: 100               #Количество опыта, которое нужно набрать перед обновлением модели.
      learning_rate: 3.0e-4          #Устанавливает шаг обучения (начальная скорость).
      beta: 5.0e-4                   #Отвечает за случайность действия, повышая разнообразие и иследованность пространства обучения.
      epsilon: 0.2                   #Порог расхождений между старой и новой политиками при обновлении.
      lambd: 0.99                    #Определяет авторитетность оценок значений во времени. Чем выше значение, тем более авторитетен набор предыдущих оценок.
      num_epoch: 3                   #Количество проходов через буфер опыта, при выполнении оптимизации.
      learning_rate_schedule: linear #Определяет, как скорость обучения изменяется с течением времени, линейно уменьшает скорость.
    network_settings:                #Определяет сетевые настройки.
      normalize: false               #Отключается нормализация входных данных.
      hidden_units: 128              #Количество нейронов в скрытых слоях сети.
      num_layers: 2                  #Количество скрытых слоев для размещения нейронов.
    reward_signals:                  #Задает сигналы о вознаграждении.
      extrinsic:
        gamma: 0.99                  #Коэффициент скидки для будущих вознаграждений.
        strength: 1.0                #Шаг для learning_rate.
    max_steps: 500000                #Общее количество шагов, которые должны быть выполнены в среде до завершения обучения.
    time_horizon: 64                 #Количество циклов ML агента, хранящихся в буфере до ввода в модель.
    summary_freq: 10000              #Количество опыта, который необходимо собрать перед созданием и отображением статистики.


```
- Decision Requester - запрашивает решение через регулярные промежутки времени и обрабатывает чередование между ними во время обучения.
- Behavior Parameters - определяет принятие объектом решений, в него указывается какой тип поведения будет использоваться: уже обученная модель или удалённый процесс обучения.


## Выводы

Я познакомился с программными средствами для создания системы машинного обучения и ее интеграции в Unity. Реализовал систему машинного обучения в связке Python - Unity.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
