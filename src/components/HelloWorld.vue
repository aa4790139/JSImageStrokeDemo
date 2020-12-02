<!--
//*************************************************************************
//    创建作者:    Harry
//    版权所有:    剑齿虎
//    开发版本:    V1.0
//    责任说明:    
//    相关说明:    
//   参考资料：https://stackoverflow.com/questions/24039599/how-to-add-stroke-outline-to-transparent-png-image-in-javascript-canvas
//*************************************************************************
-->
<template>
	<div class="hello" ref="contain">
		<h1>JS 图片描边demo</h1>
		<canvas id="canvas" ref="cvs" width="800" height="800"></canvas>
	</div>
</template>

<script>
import geom from '../../common/geom.js';
export default {
	name: 'HelloWorld',
	props: {
		msg: String
	},
	data() {
		return {
			ctx: undefined,
			canvas: undefined,
			canvases: undefined,
			img: undefined,
			imageData: undefined,
			data: undefined,
			imageData1: undefined,
			data1: undefined,
			strokeWeight: 8,
			cw: undefined,
			ch: undefined
		};
	},
	methods: {
		__Init() {
			this.canvas = document.getElementById("canvas");
			this.ctx = canvas.getContext('2d');

			// variables used in pixel manipulation
			this.canvases = [];

			// true/false function used by the edge detection method

			// the image receiving the sticker effect
			this.img = new Image();
			this.img.crossOrigin = 'anonymous';
			this.img.onload = this.__Onload;
			this.img.src = 'http://e7show-test.oss-cn-shenzhen.aliyuncs.com/Temp/image_stroke.png';
		},
		defineNonTransparent(x, y) {
			return this.data1[(y * this.cw + x) * 4 + 3] > 0;
		},
		__Onload() {
			// resize the main canvas to the image size
			this.canvas.width = this.cw = this.img.width;
			this.canvas.height = this.ch = this.img.height;

			// draw the image on the main canvas
			this.ctx.drawImage(this.img, 0, 0);

			// Move every discrete element from the main canvas to a separate canvas
			// The sticker effect is applied individually to each discrete element and
			// is done on a separate canvas for each discrete element
			while (this.moveDiscreteElementToNewCanvas()) {
				
			}

			var strokeWeight = 8;

			// add the sticker effect to all discrete elements (each canvas)
			for (var i = 0; i < this.canvases.length; i++) {
				this.addStickerEffect(this.canvases[i], strokeWeight);
				this.ctx.drawImage(this.canvases[i], 0, 0);
			}

			// redraw the original image
			//   (necessary because the sticker effect
			//    slightly intrudes on the discrete elements)
			this.ctx.drawImage(this.img, 0, 0);
		},
		addStickerEffect(canvas, strokeWeight) {
			var url = canvas.toDataURL();
			var ctx1 = canvas.getContext('2d');
			var pts = canvas.outlinePoints;
			this.addStickerLayer(ctx1, pts, strokeWeight);
			var imgx = new Image();
			imgx.onload = function() {
				ctx1.drawImage(imgx, 0, 0);
			};
			imgx.src = url;
		},
		addStickerLayer(context, points, weight) {
			this.imageData = context.getImageData(0, 0, canvas.width, canvas.height);
			this.data1 = this.imageData.data;

			var points = geom.contour(this.defineNonTransparent);

			this.defineGeomPath(context, points);
			context.lineJoin = 'round';
			context.lineCap = 'round';
			context.strokeStyle = 'white';
			context.lineWidth = weight;
			context.stroke();
		},
		// This function finds discrete elements on the image
		// (discrete elements == a group of pixels not touching
		//  another groups of pixels--e.g. each individual sprite on
		//  a spritesheet is a discreet element)
		moveDiscreteElementToNewCanvas() {
			// get the imageData of the main canvas
			this.imageData = this.ctx.getImageData(0, 0, canvas.width, canvas.height);
			this.data1 = this.imageData.data;

			// test & return if the main canvas is empty
			// Note: do this b/ geom.contour will fatal-error if canvas is empty
			var hit = false;
			for (var i = 0; i < this.data1.length; i += 4) {
				if (this.data1[i + 3] > 0) {
					hit = true;
					break;
				}
			}
			if (!hit) {
				return;
			}

			// get the point-path that outlines a discrete element
			var points = geom.contour(this.defineNonTransparent);

			// create a new canvas and append it to page
			var newCanvas = document.createElement('canvas');
			newCanvas.width = canvas.width;
			newCanvas.height = canvas.height;
			// document.body.appendChild(newCanvas);
			this.$refs.contain.appendChild(newCanvas);
			this.canvases.push(newCanvas);
			var newCtx = newCanvas.getContext('2d');

			// attach the outline points to the new canvas (needed later)
			newCanvas.outlinePoints = points;

			// draw just that element to the new canvas
			this.defineGeomPath(newCtx, points);
			newCtx.save();
			newCtx.clip();
			newCtx.drawImage(canvas, 0, 0);
			newCtx.restore();

			// remove the element from the main canvas
			this.defineGeomPath(this.ctx, points);
			this.ctx.save();
			this.ctx.clip();
			this.ctx.globalCompositeOperation = 'destination-out';
			this.ctx.clearRect(0, 0, canvas.width, canvas.height);
			this.ctx.restore();

			return true;
		},
		defineGeomPath(context, points) {
			context.beginPath();
			context.moveTo(points[0][0], points[0][1]);
			for (var i = 1; i < points.length; i++) {
				context.lineTo(points[i][0], points[i][1]);
			}
			context.lineTo(points[0][0], points[0][1]);
			context.closePath();
		}
	},
	mounted() {
		this.__Init();
	},
	created() {
		
	}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
	margin: 40px 0 0;
}
ul {
	list-style-type: none;
	padding: 0;
}
li {
	display: inline-block;
	margin: 0 10px;
}
a {
	color: #42b983;
}

.hello {
	position: absolute;
	width: 100%;
	height: 100%;
	top: 0;
	left: 0;
	overflow-y: auto;
	background-color: #b0a9a9;
}
</style>
