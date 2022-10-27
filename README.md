# lab3
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
# 2 ЛАБОРАТОРНАЯ РАБОТА. СБОР, ОБРАБОТКА И ВИЗУАЛИЗАЦИЯ ТЕСТОВОГО НАБОРА ДАННЫХ.
Отчет по лабораторной работе #2 выполнил(а):
- Маслий Илья Андреевич
- РИ211102
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity

## Задание 1
Ход работы:
Реализовать совместную работу и передачу данных в связке Python- Google-Sheets – Unity.
- В облачном сервисе google console подключить API для работы с google sheets и google drive.
![image](https://user-images.githubusercontent.com/29748577/195107335-7c979093-39b1-4fb4-a69c-a517345381e9.png)
- Реализовать запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
Генерация чисел: ![image](https://user-images.githubusercontent.com/29748577/195107706-e020459b-37e7-48e9-8e03-563cd91662ed.png)
Запись в гугл таблицу: ![image](https://user-images.githubusercontent.com/29748577/195107979-3f663f9f-0201-4c33-8485-f3b464e44fd5.png)
- Создать новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.
![image](https://user-images.githubusercontent.com/29748577/195108582-c5e38666-3fb2-4992-a507-1751fefca984.png)
- Написать функционал на Unity, в котором будет воспризводиться аудио-файл в зависимости от значения данных из таблицы.
Скриншот из Unity: ![image](https://user-images.githubusercontent.com/29748577/195109203-0de58b92-eaf3-40d8-9b99-d73acbb6c693.png)
Код скрипта:
```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
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
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1Uvav6XaYcGg3mDLhNo8hEkHhsjnl72Ar_S51Y_qFCQc/values/Лист1?key=AIzaSyD_iIQAKPc3KEGTseo2la69RLaPQ8B_gY0");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

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
