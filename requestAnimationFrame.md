#RequestAnimationFrame（RAF）帧动画函数使用
如果你想做逐帧动画的时候，你应该用这个方法。这就要求你的动画函数执行会先于浏览器重绘动作。通常来说，被调用的频率是每秒60次，但是一般会遵循W3C标准规定的频率。如果是后台标签页面，重绘频率则会大大降低。

##使用场景一：动画
RAF本身解决js动画性能，并且非常智能，防止并解决丢帧问题。当它发现无法维持60fps的频率时，它会把频率降低到30fps来保持帧数的稳定。也就是说如果上一次raf的回调执行时间过长，那么触发下一次raf回调的时间就会缩短，反之亦然，这也是为什么说由浏览器来决定执行时机性能会更好。
    (function animloop(){
    
      window.requestAnimFrame(animloop);
      
      render();
      
    })();
    
##使用场景二：函数节流
在高频率事件中，为了防止16ms内发生多次函数执行，使用raf可保证16ms内只触发一次，这既能保证流畅性也能更好的节省函数执行的开销。16ms内函数执行多次没有意义，因为显示器16ms刷新一次，多次执行并不会在界面上有任何显示。
    $box = $('#J_num2'),$point = $box.find('i');
   
    $box.on('mousemove',function(e){ 
      requestAnimationFrame(function(){ 
      $point.css({   
      top : e.pageY,   
      left : e.pageX   
      })   
      })  
    })

##使用场景三：CPU，GPU节能，更少的的cpu，gpu和内存使用量
raf的另一个特性是：如果页面不是激活状态下的话，函数会自动暂停，有效节省了CPU开销。在移动端，如果页面中有自动播放的轮播图、倒计时或使用setTimeout/setInterval来执行任务的定时器。那么当app进到后台或是锁屏后，webviewcorethread仍然持续占用CPU，导致耗电。而使用raf可以很简单的解决此类问题。

    (function(){

    var timer;
    var $txt = $('#J_num2'),
    num = 0;
    function play(){
    timer = setTimeout(function(){
      //使用raf实现非激活状态下不运行
    requestAnimationFrame(function(){
    stop();
    next();
    });
    },1000)
    }
     
    function stop(){
    clearTimeout(timer)
    }
     
    function next(){
    $txt.text(num++);
    play();
    }
    play();
    })();


##使用场景四：分帧初始化
raf的执行时间约为16.7ms，即为一帧。那么可以使用raf将页面初始化的函数进行打散到每一帧里，这样可以在初始化时降低CPU及内存开销。

很多页面，初始化加载时，CPU都会有很明显的波动，就是因为大量的操作都集中到了一点上。

举个例子：

页面中有4个模块，A、B、C、D，在页面加载时进行实例化，一般的写法类似于：

    $(function(){
    new A();
    new B();
    new C();
    new D();
    })

而使用raf可将每个模块分别初始化，即每个模块都有16ms的初始化时间

    $(function(){
    var lazyLoadList = [A,B,C,D];
    $.each(lazyloadList, function(index, module){
    window.requestAnimationFrame(function(){new module()});
    });
     
    })


##使用场景五：异步化
raf实际是一种异步化的操作，曾经setTimeout(function(){},0)一度成为解决了很多前端疑难杂症的法宝。而现在，可以用raf来代替。


##各浏览器兼容RAF方法，同setTimeout用法，如不支持RAF的直接变成使用setTimeout函数

    var lastTime = 0;
    var prefixes = 'webkit moz ms o'.split(' '); //各浏览器前缀
    
    var requestAnimationFrame = window.requestAnimationFrame;
    var cancelAnimationFrame = window.cancelAnimationFrame;
    
    var prefix;
    //通过遍历各浏览器前缀，来得到requestAnimationFrame和cancelAnimationFrame在当前浏览器的实现形式
    for( var i = 0; i < prefixes.length; i++ ) {
    	if ( requestAnimationFrame && cancelAnimationFrame ) {
    	  break;
    	}
    	prefix = prefixes[i];
    	requestAnimationFrame = requestAnimationFrame || window[ prefix + 'RequestAnimationFrame' ];
    	cancelAnimationFrame  = cancelAnimationFrame  || window[ prefix + 'CancelAnimationFrame' ] || window[ prefix + 'CancelRequestAnimationFrame' ];
    }
    
    //如果当前浏览器不支持requestAnimationFrame和cancelAnimationFrame，则会退到setTimeout
    if ( !requestAnimationFrame || !cancelAnimationFrame ) {
    	requestAnimationFrame = function( callback, element ) {
    	  var currTime = new Date().getTime();
    	  //为了使setTimteout的尽可能的接近每秒60帧的效果
    	  var timeToCall = Math.max( 0, 16 - ( currTime - lastTime ) ); 
    	  var id = window.setTimeout( function() {
    		callback( currTime + timeToCall );
    	  }, timeToCall );
    	  lastTime = currTime + timeToCall;
    	  return id;
    	};
    	
    	cancelAnimationFrame = function( id ) {
    	  window.clearTimeout( id );
    	};
    }
    
    //得到兼容各浏览器的API
    window.requestAnimationFrame = requestAnimationFrame; 
    window.cancelAnimationFrame = cancelAnimationFrame;	


RAF进行动画，下面的代码会将id为demo的div右移动到300px

    <div id="demo" style="position:absolute; width:100px; height:100px; background:#ccc; left:0; top:0;"></div>
    
    <script>
    var demo = document.getElementById('demo');
    function rander(){
    demo.style.left = parseInt(demo.style.left) + 1 + 'px'; //每一帧向右移动1px
    }
    requestAnimationFrame(function(){
    rander();
    //当超过300px后才停止
    if(parseInt(demo.style.left)<=300) requestAnimationFrame(arguments.callee);
    });
    </script>