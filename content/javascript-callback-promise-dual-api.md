+++
date = "2016-07-13T11:28:50+05:00"
title = "Двойной API для асинхронных функций"
slug = "javascript-callback-promise-dual-api"
draft = false
type = "post"
aliases = [
    "post/javascript-callback-promise-dual-api"
]
tags = [ "javascript", "callback", "promise", "node.js" ]
+++

Когда пишешь публичные асинхронные методы на **JavaScript**, желательно придерживаться двух простых правил:

1. Если метод может вернуть ошибку в колбэке, то ошибка должна идти первым аргументом. Часто, даже если метод никогда не возвращает ошибку, первым аргументом передают `null` для того, чтобы унифицировать все асинхронные вызовы;
2. Если в метод не передана колбэк-функция, то метод должен вернуть Promise.

<!--more-->

Если с первым правилом все достаточно просто, то для удобного выполнения второго, в нашей команде родилось короткое однострочное выражение, которые мы повсеместно используем:

```Javascript
const d = (typeof cb !== "function") ? new Promise((f, r) => (cb = (e, d) => e != null ? r(e) : f(d))) : null;
```

Покажу, как водится, на примере таймера:

```Javascript
const asyncMethod = (timeOut, cb) => {
    // Если cb не является функцией, создаем Promise и модифицируем cb для использования далее.
    const d = (typeof cb !== "function") ? new Promise((f, r) => (cb = (e, d) => e != null ? r(e) : f(d))) : null;

    // Используем cb как обычный колбэк
    setTimeout(cb, timeOut);

    // Возвращаем либо Promise, если колбэка не было, либо null
    return d;
};

// Используем в классическом стиле:
asyncMethod(1000, () => {
    console.log("Classic timeout reached!");
});

// Используем через Promise
asyncMethod(1000).then(() => console.log("Promise timeout reached!"));
```

Таким вот простым способом, можно улучшить публичный API в части асинхронных методов.