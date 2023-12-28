## Комп'ютерні системи імітаційного моделювання
## СПм-22-4, **Шаманов Дмитро**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo


### Варіант 22(9), модель у середовищі NetLogo:
[Sheperds](http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Biology/Shepherds.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Поділити овець на два різних стада, відповідно і пастухів на дві різні організації.** 

Пастухи повинні збирати тільки "своїх" овець.

Створено атрибут для овець та пастухів **group**, який відповідає за приналежність до першої чи другої групи (організації):
<pre>
  shepherds-own
[
  group
  carried-sheep         ;; the sheep I'm carrying (or nobody if I'm not carrying in)
  found-herd?           ;; becomes true when I find a herd to drop it in
]

sheep-own[
  group
]

</pre>

Змінено процедуру **setup**. При запуску випадково дається група вівці та пастуху. Якщо група 1, то вівця буде білою, а пастух чорним. Якщо 2, то вівця буде коричневою а пастух жовтим.
<pre>
  create-sheep num-sheep
  [ 
      set group one-of[1 2]
      ifelse group = 1 [ set color white] [ set color black ]
      set size 1.5  ;; easier to see
      setxy random-xcor random-ycor ]
  create-shepherds num-shepherds
  [ 
      set group one-of[1 2]
      ifelse group = 1 [ set color yellow] [ set color red ]
      set size 1.5  ;; easier to see
      set carried-sheep nobody
      set found-herd? false
      setxy random-xcor random-ycor ]
</pre>

Тепер пастух обирає тільки ту вівцю, яка відповідає його групі. І "кладе" вівцю у стадо тільки поруч з вівцею тієї ж групи.
<pre>
  to search-for-sheep ;; shepherds procedure
  set carried-sheep one-of sheep-here with [not hidden? and group = [group] of myself]
 
  if (carried-sheep != nobody and group = [group] of carried-sheep)
    [
      ask carried-sheep
        [ hide-turtle ]  ;; make the sheep invisible to other shepherds
           ;; turn shepherd blue while carrying sheep
      fd 1 ]
end

to find-new-herd ;; shepherds procedure
  if any? sheep-here with [not hidden? and group = [group] of myself]
    [ set found-herd? true ]
end
</pre>
                  
**Додати відключаєму можливість збирати "чужих" вівець, які після потрапляння до нового стада змінюють свою приналежність.**
Додано відповідний *switch* **can-steal-sheep**:
![1](1.png)

І змінено поведінку пастухів:
<pre>
set carried-sheep one-of sheep-here with [(not hidden? and (group = [group] of myself or steal-sheep))]
</pre>



**Змінено алгоритм вибору вівці пастухом**
Тепер пастух перевіряє всіх овець на патчі поки не знайде вільну, якщо вона є.

  <pre>
to search-for-sheep ;; shepherds procedure
  let suitable-sheep nobody
  ask sheep-here [
  if (not hidden? and (group = [group] of myself or steal-sheep)) [
    set suitable-sheep self
  ]
]
  if suitable-sheep != nobody [ set carried-sheep suitable-sheep] 
 
  if (carried-sheep != nobody)
    [
      ask carried-sheep
        [ hide-turtle ]  ;; make the sheep invisible to other shepherds
      fd 1
    ]
end

  </pre>

![Робота моделі](2.png)
![Робота моделі](3.png)
<br>
  
## Обчислювальні експерименти
### 1. Вплив кількості овець на ефективність випасання
Досліджується значення ефективності пастухів відносно кількості овець. Експерименти проводяться при налаштуваннях за замовчуванням, протягом 2000 тактів. Початкова кількість овець - 100, кількість пастухів - 30. Експерименти проводяться при 100-240 вівцях, з кроком 20, усього 8 симуляцій. Інші керуючі параметри мають значення за замовчуванням:

1. кількість пастухів - 30.
2. швидкість вівці - 0.02

<table>
<thead>
<tr><th>Номер симуляції</th><th>Кількість овець</th><th>Початкова ефективність</th><th>Кінцева ефективність</th></tr>
</thead>
<tbody>
<tr><td>1</td><td>100</td><td>76.49</td><td>82.2</td></tr>
<tr><td>2</td><td>120</td><td>73.74</td><td>80.73</td></tr>
<tr><td>3</td><td>140</td><td>69.22</td><td>78.67</td></tr>
<tr><td>4</td><td>160</td><td>65</td><td>76.4</td></tr>
<tr><td>5</td><td>180</td><td>61.37</td><td>74.61</td></tr>
<tr><td>6</td><td>200</td><td>59.02</td><td>72.54</td></tr>
<tr><td>7</td><td>220</td><td>54.64</td><td>71.32</td></tr>
<tr><td>8</td><td>240</td><td>53.69</td><td>70.11</td></tr>
</tbody>
</table>

![4](4.png)

З огляду на таблицю, можна сказати, що при збільшенні кількості овець, падає як початкова ефективність, і темпи спадання - приблизно 4 відсотки на старті та 2 відсотки в кінці на кожні додаткові 20 овець.
