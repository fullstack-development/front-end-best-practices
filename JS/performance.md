1. **Еще раз: В проектах на jquery или чем-либо, где поиск DOM-элементов идет по селекторам - надо кэшировать все найденные элементы**;
    > ```javascript
    >  $foodItem = $('#shopping-list li');
    >
    >  $foodItem.click(function() {
    >    ... do something ...
    >  });
    >```

2. **Все операции с DOM-ом лучше сначала проводить на виртуальных элементах, а потом сразу целиком вставлять в документ страницы**;
    > ```javascript
    >  for (var i=0; i < items.length; i++){ 
    >    var item = document.createElement("li");
    >    item.appendChild(document.createTextNode("Option " + i);
    >    list.appendChild(item); 
    >  }
    >```
    >здесь в цикле в DOM внедряется каждый элемент массива один за одни, в итоге имеем кучу циклов перерисовки
    >
    >```javascript
    >  var fragment = document.createDocumentFragment();
    >  for (var i=0; i < items.length; i++){
    >    var item = document.createElement("li"); 
    >    item.appendChild(document.createTextNode("Option " + i);
    >    fragment.appendChild(item); 
    >  } 
    >  list.appendChild(fragment);
    >```
    >здесь мы в цикле вносим элемент в виртуальный элемент, а затем уже целиком за раз это внедряем в реальный DOM и получаем один цикл перерисовки

3. **Избегать сложных селекторов**;
    > В зависимости от типа и сложности css-селектора зависит производительность рендеринга и анимации сайта. Например использование в `transition` селектора по атрибуту (например `input[type="text"] { transform: translateX(50%)` } теоретически может замедлить саму анимацию. Те же правила экстраполируются на всю страницу. Чем сложнее css-селекторы (много вложенности и нетривиальных селекторов с необходимостью часто и много считать приоритеты), тем сложнее браузеру рендерить страницу, т.к. приходится применять стили согласованно между друг другом и анализировать DOM-деревл, что, конечно, не самая простая задача с точки зрения производительности сама по себе.

***Полезные материалы:***
  
  1. [Perf.Rocks](https://perf.rocks/);
  2. [calendar](https://calendar.perfplanet.com/2014/);
  3. [PageSpeed Insights Rules](https://developers.google.com/speed/docs/insights/rules);
  4. [avascriptrocks](http://javascriptrocks.com/).