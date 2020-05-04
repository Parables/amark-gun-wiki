Here are some example commands I have written, maybe they'll be useful. Real documentation will come as more is demanded:

 > Note: This system is very alpha, and may have unannounced breaking changes, fixes, etc.

```html
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/gun/examples/style.css">
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/gun/examples/jquery.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/meta.js"></script>
<script>$(function(){
// on fires when shortcut keydowns or on touch after command selected and then touchdown
meta.edit({name: "Add", combo: ['A']});
meta.edit({name: "Row", combo: ['A', 'R'],
    on: function(eve){
        meta.tap().append('<div class="hold center" style="min-height: 9em; padding: 2%;">');
    }
});
meta.edit({name: "Columns", combo: ['A','C'],
    on: function(eve){
        var on = meta.tap(), tmp, c;
        var html = '<div class="unit col" style="min-height: 9em; padding: 2%;"></div>';
        if(!on.children('.col').length){ html += html }
        c = (tmp = on.append(html).children('.col')).length;
        tmp.each(function(){
            $(this).css('width', (100/c)+'%');
        })
    }
});
meta.edit({name: "Text", combo: ['A','T'],
    on: function(eve){
        meta.tap().append('<p contenteditable="true">Text</p>');
    }
});
meta.edit({name: "Move", combo: ['M']});
;(function(){
    $(document).on('click', function(){
        var tmp = $('.meta-on');
        if(!tmp.length){ return }
        tmp.removeClass('meta-on');
    })
    meta.edit({combo: [38], // up
        on: function(eve){
            var on = meta.tap().removeClass('meta-on');
            on = on.prev().or(on.parent()).or(on);
            on.addClass('meta-on');
        }, up: function(){ 
        }
    });
    meta.edit({combo: [40], // down
        on: function(eve){
            var on = meta.tap().removeClass('meta-on');
            on = on.next().or(on.children().first()).or(on);
            on.addClass('meta-on');
        }, up: function(){ 
        }
    });
    meta.edit({combo: [39], // right
        on: function(eve){
            var on = meta.tap().removeClass('meta-on');
            on = on.children().first().or(on.next()).or(on.parent()).or(on);
            on.addClass('meta-on');
        }, up: function(){ 
        }
    });
    meta.edit({combo: [37], // left
        on: function(eve){
            var on = meta.tap().removeClass('meta-on');
            on = on.parent().or(on);
            on.addClass('meta-on');
        }, up: function(){ 
        }
    });
}());
meta.edit({name: "Turn", combo: ['T']});
meta.edit({name: "Size", combo: ['S']});
meta.edit({name: "X", combo: ['S','X'],
    on: function(eve){
        var on = meta.tap(), was = on.width();
        $(document).on('mousemove.tmp', function(eve){
            var be = was + ((eve.pageX||0) - was);
            on.css({'max-width': be, width: '100%'});
        })
    }, up: function(){ $(document).off('mousemove.tmp') }
});
meta.edit({name: "Y", combo: ['S','Y'],
    on: function(eve){
        var on = meta.tap(), was = on.height();
        $(document).on('mousemove.tmp', function(eve){
            var be = was + ((eve.pageY||0) - was);
            on.css({'min-height': be});
        })
    }, up: function(){ $(document).off('mousemove.tmp') }
});
meta.edit({name: "Fill", combo: ['F'],
    on: function(eve){
        var on = meta.tap();
        meta.ask('Color name, code, or URL?', function(color){
            var css = on.closest('p').length? 'color' : 'background';
            on.css(css, color);
        });
    }
});
});</script>
</body>
</html>
```