> Письма верстаю **табличной раскладкой** с **inline-CSS**, фиксированной шириной ~**600px**, без JS.  
> Использую **MJML** для разработки, в CI компилирую в “bulletproof” HTML. Делаю **responsive** через поддерживаемые медиа-запросы, **тёмную тему**, **bulletproof-кнопки** (VML для Outlook), альтернативу **text/plain**, **preheader**, alt-тексты, и стараюсь держать размер <100KB.  
> Шаблоны параметризую (Jinja2/Handlebars), локализация, A/B, иду через наш нотификационный сервис: событие в Kafka → воркер рендерит → SMTP/ESP. Логи отправок и метрики в Prometheus/Grafana, ошибки в Sentry.

---

## Практические принципы

- **Макет**: таблицы `<table>`, не flex/grid. Контейнер 600–640px.
    
- **CSS**: только **inline**; минимум `<style>` (Outlook режет). Без внешних шрифтов, fallback: `Arial, sans-serif`.
    
- **Кнопки**: “bulletproof” (HTML + VML для Outlook).
    
- **Изображения**: абсолютные URL из CDN, `alt`, `width/height` атрибуты, не base64. Не класть текст в картинки.
    
- **Responsive**: простые медиа-запросы + fluid-изображения (`max-width:100%`).
    
- **Dark mode**: инверсия цветов через поддерживаемые селекторы, дублирование логотипа с тёмной версией при необходимости.
    
- **Доступность**: читабельный контраст, `role="presentation"` у таблиц, `aria-label` у ссылок-кнопок.
    
- **Контент**: **preheader** (скрытый текст), версия **text/plain** (multipart/alternative).
    
- **Юридически/доставляемость**: обязательные реквизиты отправителя, **unsubscribe**, короткие темы, `List-Unsubscribe`, SPF/DKIM/DMARC настроены.
    
- **Ограничения**: Gmail обрезает >102KB; избегать форм и сложных CSS; анимации — только GIF.
    
- **Тесты**: прогон через рендер-превью (Litmus/Email on Acid) и тест-ящики (Gmail/Outlook/Apple Mail/Yahoo).
    

---

## Мини-пайплайн (как у нас в Т-Банке)

1. Сервис публикует событие `email.send` в **Kafka**.
    
2. Воркер берёт **шаблон** (MJML/HTML) + данные, рендерит (Jinja2), инлайнит стили (Premailer).
    
3. Отправка через SMTP/ESP (иногда через провайдера с доменной подписью).
    
4. Лог отправки в Postgres (status, message-id), метрики (deliveries, bounces) в Prometheus; ошибки → Sentry.
    
5. Лимитирование/идемпотентность по `message_key`.
    

---

## Пример: MJML → HTML

### MJML (читабельно для разработки)

```
<mjml>
  <mj-body background-color="#f5f6f8">
    <mj-section>
      <mj-column>
        <mj-text font-size="20px" font-weight="700">Подтверждение платежа</mj-text>
        <mj-text>Здравствуйте, {{ user_name }}! Ваш платёж №{{ order_id }} на сумму {{ amount }} ₽ выполнен.</mj-text>
        <mj-button href="{{ action_url }}" background-color="#0a66c2">
          Посмотреть в приложении
        </mj-button>
        <mj-text font-size="12px" color="#6b7280">Если это были не вы — немедленно свяжитесь с поддержкой.</mj-text>
      </mj-column>
    </mj-section>
    <mj-section>
      <mj-column>
        <mj-text font-size="11px" color="#9ca3af">
          © Т-Банк · Это письмо отправлено автоматически. <a href="{{ unsub_url }}">Отписаться</a>
        </mj-text>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>

```

В CI компилируем в “табличный” HTML и **инлайн-CSS** — пригодный для Outlook.

---

## Фрагмент “bulletproof” кнопки (HTML+VML для Outlook)

```
<!--[if mso]>
<v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" href="{{ action_url }}" arcsize="8%" fillcolor="#0a66c2" stroke="f">
  <v:textbox inset="0,0,0,0">
    <center style="color:#ffffff;font-family:Arial;font-size:16px;">Посмотреть в приложении</center>
  </v:textbox>
</v:roundrect>
<![endif]-->
<![if !mso]><!-- -->
<a href="{{ action_url }}" 
   style="background:#0a66c2;color:#fff;text-decoration:none;padding:12px 20px;border-radius:6px;display:inline-block;font-family:Arial;font-size:16px;">
  Посмотреть в приложении
</a>
<!-- <![endif]>

```

---

## Тёмная тема (минимальный приём)
`
```
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
<style>
@media (prefers-color-scheme: dark) {
  .email-body { background:#0b0f14 !important; color:#e5e7eb !important; }
  a { color:#8ab4f8 !important; }
}
</style>

```

---

## Пара “граблей” и как обходили

- **Outlook ломает отступы/шрифты** → табличная верстка, VML для кнопок, фиксируем `line-height`.
    
- **Gmail режет CSS в `<head>`** → инлайнить всё критичное.
    
- **Капча/формы в письмах** → избегать; делаем CTA-кнопку.
    
- **Размер >102KB** → ужимаем стили/тексты/удаляем лишние комментарии.
    
- **Трекинг-пиксель и UTM** → учесть privacy-режимы (Apple MPP), метрики смотреть с осторожностью.
    

---

## Проверочный чек-лист перед релизом

-  HTML таблицами, ширина 600–640px, inline-CSS
    
-  Alt-тексты, `width/height` у картинок
    
-  Text/plain альтернатива, корректный `preheader`
    
-  Кнопки совместимы с Outlook (VML)
    
-  Dark-mode минимум настроен
    
-  Unsubscribe и адрес отправителя присутствуют
    
-  DKIM/SPF/DMARC ok, размер < 100KB
    
-  Прогон через реальные ящики + рендер-тесты