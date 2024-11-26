# Валидация

## Доступные правила
> Нажмите на заголовок, что бы увидеть описание.

<details>
<summary><b>required</b></summary>

Проверяемое поле должно присутствовать во входных данные и не должно быть пустым. Поле считается "пустым" если:

- Значение равно `null`
- Значение пустая строка
- Массив не пустой
- Файл загружен с ошибками
</details>

<details>
<summary><b>required_if:another_field,value1,value2...</b></summary>

Проверяемое поле должно присутствовать во входных данные и не должно быть пустым, если `another_field` <b>равно</b> любому из указанных значений.
</details>

<details>
<summary><b>required_unless:another_field,value1,value2...</b></summary>

Проверяемое поле должно присутствовать во входных данные и не должно быть пустым, если `another_field` <b>не равно</b> любому из указанных значений.
</details>

<details>
<summary><b>email:mx</b></summary>

Проверяемое поле должно быть действительным email-адресом. Проверка осуществляется через PHP функцию `filter_var`. Дополнительный параметр `mx`, позволяет проверить наличие mx записей домена, используя PHP функцию `getmxrr`.
</details>

<details>
<summary><b>default:value</b></summary>

Это специальное правило ничего не подтверждает, оно устанавливает для поля значение по умолчанию в случае, если поле отсутствует или его значение пустое.
</details>

<details>
<summary><b>lowercase</b></summary>

Проверяемое поле должно содержать строку в нижнем регистре.
</details>

<details>
<summary><b>uppercase</b></summary>

Проверяемое поле должно содержать строку в верхнем регистре.
</details>

<details>
<summary><b>same:another_field</b></summary>

Значение проверяемого поля должно совпадать с `another_field`.
</details>

<details>
<summary><b>different:another_field</b></summary>

Значение проверяемого поля не должно совпадать с `another_field`.
</details>
