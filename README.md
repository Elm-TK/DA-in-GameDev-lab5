# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Ельмуратов Темирлан Кеулимжаевич
- РИ-210913

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

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity


## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity.
Ход работы:

- В облачном сервисе google console подключить API для работы с google sheets и google drive.

![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/1.png)
![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/2.png)


- Реализовать запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.

![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/3.png)


- Создать новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.

![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/4.png)


- Написать функционал на Unity, в котором будет воспризводиться аудио-файл в зависимости от значения данных из таблицы.

![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/51.png)

## Задание 2
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1. 
Ход работы:
```python
import numpy as np
import gspread


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
    db = (1.0 / num) * ((prediction - y).sum())
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times):
    for i in range(times):
        a, b = optimize(a, b, x, y)
    return a, b


gs = gspread.service_account(filename="corded-woodland-365110-a012dbb3b20c.json")
sh = gs.open("UnitySheets").get_worksheet(0)

x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
x = np.array(x)
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 199]
y = np.array(y)

a = np.random.rand(1)
print(a)
b = np.random.rand(1)
print(b)
Lr = 0.000001

a, b = iterate(a, b, x, y, 1)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)

for i in range(len(x)):
    sh.update(('A' + str(i + 1)), str(i + 1))
    sh.update(('B' + str(i + 1)), str(x[i]))
    sh.update(('C' + str(i + 1)), str(prediction[i]))

```
![](https://github.com/Elm-TK/DA-in-GameDev-lab2/blob/main/6.png)




## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2.
```cs
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

    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    void Update()
    {
        if (i == dataSet.Count || statusStart || !dataSet.ContainsKey("Mon_1"))
            return;

        var value = dataSet["Mon_" + i];

        switch (value)
        {
            case < 5 / 10:
                StartCoroutine(PlaySelectAudio(GoodSpeak, 4));
                break;
            case >= 5 / 10 and < 1:
                StartCoroutine(PlaySelectAudio(NormalSpeak, 4));
                break;
            case >= 1:
                StartCoroutine(PlaySelectAudio(BadSpeak, 4));
                break;
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1UdLCXKWZux0VFoIQcaGnCTQhuCp-R5EmmQ-MVb-fOzw/values/Лист1?key=AIzaSyBnBxL15m9oVvkoYXEMVaubzTXE1VecAYU");
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
    IEnumerator PlaySelectAudio(AudioClip clip, int time)
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = clip;
        selectAudio.Play();
        yield return new WaitForSeconds(time);
        statusStart = false;
        i++;
    }
}

```

## Выводы


