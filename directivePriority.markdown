## directive priority

Angular.js compiles directives in order of priority. Higher number = Higher
priority.

Angular also offers a second way to control compilation: the `terminal`
attribute of the directive definition object.

Directives are compiled from high to low priority, but if a directive is
encountered that has terminal set to true, then the compilation stops after
processing the rest of the directives at the current priority.

## real world use

This is a directive that sets certain options like delays before updating the
model, etc. This would normally be straightforward, except that we also have a
directive named 'decimal-places' that formats the number, and requires ngModel
to work its magic.

```
# index.html:
&lt;input instant-edit="MyObj.myField" decimal-places=2 />
```

```
# instantEdit.js

var app = angular.module('MyApp');
app.directive('instantEdit', ['$compile', function($compile) {
    return {
        restrict: 'A',
        // Set a higher priority for this directive to ensure that ng-model is
        // compiled first. This ensures that any other directives that depend
        // on ng-model will work.
        priority: 1500,
        // This stops the compilation once it reaches this directive.
        terminal: true,
        compile: function(element, attrs) {
            element.attr('ng-model', element.attr('instant-edit'));
            element.attr('ng-model-options',  "{ updateOn: 'default blur', debounce: {'default': 1000, 'blur': 0} }");
            // Very important to prevent infinite loops
            element.removeAttr('instant-edit');
            return {
                pre: function() {},
                post: function(cScope, cElement) {
                    $compile(cElement)(cScope);
                }
            };
        }
    }
}]);
```

In this example, we create a directive that sets up an input box the way we
want them to work: saving on blur, or when the user doesn't type for 1000
milliseconds.  By setting the priority very high, it compiles this directive
first, and stops after doing the compile.  This prevents any other directives
on the element from compiling at this point.

Once this has compiled, the ng-model directive compiles at priority 1000, and
then our decimal-places directive compiles at priority 0, with ng-model already
available on the current element scope.
