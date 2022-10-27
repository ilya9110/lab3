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
| Задание 2 | # | 20 |
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
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1
Ход работы:

```py

import gspread
import numpy as np
import matplotlib.pyplot as plt

gc = gspread.service_account(filename='unity-365207-9cc8948d5ec2.json')
sh = gc.open('UnitySheets')
priceX = np.random.randint(2000, 10000, 11)
priceX = np.array(priceX)
priceY = np.random.randint(2000, 10000, 11)
priceY = np.array(priceY)
mon = list(range(1, 11))


def model(a, b, x):
  return a * x + b


def loss_function(a, b, x, y):
  num = len(x)
  prediction = model(a, b, x)
  return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
  num = len(x)
  prediction = model(a, b, x)
  da = (1.0 / num) * ((prediction - y) * x).sum()
  db = (1.0 / num) * (prediction - y).sum()
  a = a - Lr * da
  b = b - Lr * db
  return a, b


def iterate (a, b, x, y, times):
  for i in range(times):
    a, b = optimize(a, b, x, y)
  return a, b


x = [2, 11, 13, 16, 25, 36, 49, 56, 76, 84, 99]
x = np.array(x)
y = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
y = np.array(y)

a = np.random.rand(1)
b = np.random.rand(1)
Lr = 0.000001

a, b = iterate(a, b, priceX, priceY, 1)
prediction = model(a, b, priceX)
loss = loss_function(a, b, priceX, priceY)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)


i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = ((prediction[i-1]-prediction[i-2])/prediction[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(prediction[i-1]//1))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(prediction)


```
Скриншот GoogleSheets: 
![image](https://user-images.githubusercontent.com/29748577/195120159-0787b0bf-1bc2-4ed6-bb0f-22f175d4d5a7.png)
Скриншот PyCharm:![image](https://user-images.githubusercontent.com/29748577/195124478-bc66faae-fd3f-4c8d-aa6a-adae01fb5895.png)


## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2
Ход работы:

```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip GoodSpeak;
    public AudioClip NormalSpeak;
    public AudioClip BadSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;


    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (CheckInflation(int.MinValue, 10))
        {
            StartCoroutine(PlaySelectAudioMode(GoodSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (CheckInflation(10, 100))
        {
            StartCoroutine(PlaySelectAudioMode(NormalSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (CheckInflation(100, int.MaxValue))
        {
            StartCoroutine(PlaySelectAudioMode(BadSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    bool CheckInflation(int min, int max)
    {
        var value = dataSet["Mon_" + i.ToString()];
        return value >= min & value <= max & statusStart == false & i != dataSet.Count;
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1Iji8SOdY2JcdGTF19Nd0O2aQLkdRdeoAxntz2xZPICE/values/Лист1?key=AIzaSyAxo_3QPnebOrbOnnwIJxtKjy83Wz9t4lE");
        yield return curentResp.SendWebRequest();
        string RawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(RawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioMode(AudioClip mode)
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = mode;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

## Выводы

Я научился реализовывать функционал, позволяющий генерировать стоимость товара (ресурса или игрового объекта) в виде набора данных. Научился на движке Unity реализовывать функционал, позволяющий воспроизводить аудио-файлы со звуковой информацией в зависимости от значений входного набора данных из таблицы. 

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
