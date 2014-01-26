---
title: '尝试将块化的内容在页面上进行旋转后绘制'
layout: default
---
用于尝试在页面上进行旋转绘制相应的图片

-方式一：直接绘制在内存的canvas中，然后直接旋转进行绘制
-方式二：直接单张绘制在canvas中

	<head>
		<meta charset="UTF-8">
		<title>Document</title>
		<script>
			document.addEventListener("DOMContentLoaded", function(){
				drawTmpCvs();
				drawCvsEach();
			});

			// draw each block
			var drawCvsEach = function() {
				var canvas = document.getElementById("myCanvas");
				var ctx = canvas.getContext("2d");

				ctx.save();
				ctx.translate( 20, 20 );
				ctx.rotate( 30 * Math.PI / 180 );
				ctx.fillRect( 0, 0, 50, 50 );
				ctx.restore();

				ctx.save();
				ctx.translate( 20, 20 );
				ctx.rotate( 30 * Math.PI / 180 );
				ctx.globalAlpha = 0.6;
				ctx.fillStyle = "#ff0000";
				ctx.fillRect( 0, 40, 50, 10 );
			}

			// draw image in memory canvas
			var drawTmpCvs = function(){
				var canvas = document.getElementById("templateCanvas");
				var ctx = canvas.getContext("2d");

				var cacheCVS = document.createElement('canvas');
				var cacheCtx = cacheCVS.getContext( "2d" );
				cacheCVS.width = 50;
				cacheCVS.height = 50;
				cacheCtx.fillRect( 0, 0, 50, 50 );

				cacheCtx.save();
				cacheCtx.globalAlpha = 0.6;
				cacheCtx.fillStyle = "#ff0000";
				cacheCtx.fillRect( 0, 40, 50, 10 );
				cacheCtx.restore();

				ctx.save();
				ctx.translate( 20, 20 );
				ctx.rotate( 30 * Math.PI / 180 );
				ctx.drawImage( cacheCVS, 0, 0 );
				ctx.restore();
			};
		</script>
	</head>
	<body>
		<canvas id='myCanvas' width="100" height="100" style="background-color: #00ff00"></canvas>
		<canvas id="templateCanvas" width="100" height="100" style="background-color: #00ff00"></canvas>
	</body>
	</html>