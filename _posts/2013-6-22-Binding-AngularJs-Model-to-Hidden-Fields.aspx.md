---
layout: post
title: Binding AngularJs Model to Hidden Fields
category: AngularJs
---

For whatever reason AngularJS ignores hidden fields when using the **ng-model** directive. This means the value of the hidden field is not updated when the model changes, so a form submit will post empty hidden fields, even though the model has the correct values.

 It seems that the only way to solve this problem is to do it manually, in this case with my own directive

  
```csharp
.directive('ngUpdateHidden',function() {
    return function(scope, el, attr) {
        var model = attr['ngModel'];
        scope.$watch(model, function(nv) {
            el.val(nv);
        });

    };
})
```
  This directive will monitor changes of the scope field designated by ng-model and then updates the hidden field.

  
```csharp
<input type="hidden" name="item.Name" ng-model="item.Name" ng-update-hidden />
```
 
