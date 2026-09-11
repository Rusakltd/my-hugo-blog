---
title: Настройка проброса UTM-меток через GTM
date: 2026-09-11
tags:
  - gtm
  - web-analytics
summary: Скрипт для проброса UTM-меток через Google Tag Manager из URL в кнопку на посадочной странице
---
## Проблема

В Steam аналитика строится на основе UTM-меток, проблема возникает когда используется посадочная страница. UTM-метки каждой рекламной кампании надо перенести в кнопку на сайте иначе они останутся только в URL самой посадочной страницы. Ниже как это сделать с помощью одного тега и триггера в Google Tag Manager.

## Настройки тега

Закидываем код в кастомный html-тег

```javascript
<script>
(function() {
  var domainsToDecorate = [
          'store.steampowered.com', //add or remove domains (without https or trailing slash)
      ],
      queryParams = [
          'utm_medium', //add or remove query parameters you want to transfer
          'utm_source',
          'utm_campaign',
          'utm_content',
          'utm_term'
      ]
  // do not edit anything below this line
  var links = document.querySelectorAll('a'); 

// check if links contain domain from the domainsToDecorate array and then decorates
  for (var linkIndex = 0; linkIndex < links.length; linkIndex++) {
      for (var domainIndex = 0; domainIndex < domainsToDecorate.length; domainIndex++) { 
          if (links[linkIndex].href.indexOf(domainsToDecorate[domainIndex]) > -1 && links[linkIndex].href.indexOf("#") === -1) {
              links[linkIndex].href = decorateUrl(links[linkIndex].href);
          }
      }
  }
// decorates the URL with query params
  function decorateUrl(urlToDecorate) {
      urlToDecorate = (urlToDecorate.indexOf('?') === -1) ? urlToDecorate + '?' : urlToDecorate + '&';
      var collectedQueryParams = [];
      for (var queryIndex = 0; queryIndex < queryParams.length; queryIndex++) {
          if (getQueryParam(queryParams[queryIndex])) {
              collectedQueryParams.push(queryParams[queryIndex] + '=' + getQueryParam(queryParams[queryIndex]))
          }
      }
      return urlToDecorate + collectedQueryParams.join('&');
  }

  // borrowed from https://stackoverflow.com/questions/831030/
  // a function that retrieves the value of a query parameter
  function getQueryParam(name) {
      if (name = (new RegExp('[?&]' + encodeURIComponent(name) + '=([^&]*)')).exec(window.location.search))
          return decodeURIComponent(name[1]);
  }

})();
</script>
```

## Настройки триггера

- Trigger Type: Page View - Dom Ready
- Fire on «Some DOM ready Events»: 
  `{{Page URL}}` — `matches RegEx (ignore case)` — `utm_source=|utm_medium=|utm_campaign=|utm_term=|utm_content=`