## Passing Angular scope attached variables into <template is="auto-binding"></template> element. Check Plnkr https://plnkr.co/edit/Hm4oo3b5K75ptmPDqNY5?p=preview

Using template elements may prove to be extremely useful in many cases. 

They give you abiliy to insert inert and lazy loaded (meaning, e.g. images, scripts inside the template tags are not loadded on page 

load).

On top of this, polymer has brought auto-binding which gives you the power of databinding.

```html
<template is="auto-binding" id="greeter">
 Hello {{username}}!
</template>
```


So in the script, we can bind username like below.
```javascript
var tmpl = document.querySelector('#tmpl');
tmpl.username = 'Ganesh';
```

So on the page, you will automatically see Hello Ganesh!

So with angularjs it's not that easy.
```javascript
var app = angular.module('app', []);
app.controller('ctrl', function($scope){
	$scope.model = {
			username:'Ganesh' 	
		}

});
```


Now what you want to do is access this $scope.model inside the template as below,
```html
<template is="auto-binding" id="greeter">
 Hello {{model.username}}!
</template>
```

But unfortunately, does't work!! Sad !!!

## Solution:
I developed a simple but smart directive which does this for you.

```javascript
app.directive('copyFromNg', function($parse) {
  return {
    restrict: 'AE',
    link: function(scope, element, attrs) {
      //var el = document.querySelector('#' + attrs.id);
      var el = element[0];
      window.addEventListener('WebComponentsReady', function() {
        var keys = Object.keys(this.attrs.$attr);
        var avoidKeys = ['copyFromNg', 'is', 'id'];
        for (var i = 0; i < keys.length; i++) {
          var key = keys[i];
          if (avoidKeys.indexOf(key) > -1) {
            continue;
          }
          var value = $parse(this.attrs[key])(this.context) || this.attrs.$attr[key];
          el[key] = value;
        }
      }.bind({context:scope, attrs:attrs, elem:el}));
    },
    template: ''
  };
});
```

Now let's apply this angular directive to our template:
```html
<template is="auto-binding" id="greeter"  information="model" copy-from-ng   >
 Hello {{information.username}}!
</template>

<!-- For polymer 1.x new version -->
<template is="dom-bind" id="greeter"  information="model" copy-from-ng   >
 Hello {{information.username}}!
</template>
```


Bravo!! you see Hello Ganesh!, this time from angular scope.

