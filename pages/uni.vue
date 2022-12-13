<template>
	<section class="page-wrapper">
		<v-select 
		label="نوع" 
		:items="modes" 
		solo 
		outlined 
		v-model="mode"
		@change="changeMode"
		item-text="text"
		item-value="value"></v-select>
		<div class="flex-wrapper">
			<label class="attach">

				<div class="upload">
										ورودی
										
					<div class="upload-box">
						<canvas id="mycanvas"></canvas>
						<img class="uploaded-img" id="myImg" :src="cover1src">
					</div>
				</div>
				<input ref="cover1" class="input-file" accept="image/*" type="file"
						@change="uploadCover1">
			</label>
			<label class="attach">
				<div class="upload">
										نتیجه
										
					<div class="upload-box">
						<canvas id="resultCanvas"></canvas>
					</div>
				</div>
			</label>
		</div>
	</section>
</template>

<script>

	export default {
		name: "general",
		scrollToTop: true,
		data() {
			return {
				cover1src: '',
				modes:[
					{text: 'نگاتیو', value: 'negative'},
					{text: 'خاکستری', value: 'grayScale'},
					{text: 'روشن‌تر', value: 'brighter'},
					{text: 'تیره‌تر', value: 'darker'},
					{text: 'تعویض آبی و قرمز', value: 'R&B'},
				],
				mode: 'negative',
				totalRed: 0,
				totalGreen: 0,
				totalBlue: 0
			}
		},
		methods: {
			changeImage(){
				// initialize input canvas 
				let canvas = document.getElementById("mycanvas");
				let context = canvas.getContext("2d")
				let imgSrc = document.getElementById("myImg")
				context.drawImage(imgSrc,0,0,context.canvas.width,context.canvas.height)
				let imageData = context.getImageData(0,0,context.canvas.width,context.canvas.height);
				let pixels = imageData.data;

				// initialize result canvas 
				let resultCanvas = document.getElementById("resultCanvas");
				let resultContext = resultCanvas.getContext("2d")
				resultContext.drawImage(imgSrc,0,0,resultContext.canvas.width,resultContext.canvas.height)
				let resultImageData = resultContext.getImageData(0,0,resultContext.canvas.width,resultContext.canvas.height);
				let resultPixels = []
				resultPixels = resultImageData.data;

				// now proccessing ... 
				var init = 1;
				while(init < context.canvas.height){
				for(var j = 0; j < 5; j++)
				{
				var row    = (init + j) * context.canvas.width * 4;
				for(var i = 0; i < context.canvas.width * 4; i += 4)
				{
					this.addRGB(pixels[row + i],pixels[row + i + 1],pixels[row + i + 2])
					if(this.mode === 'negative'){
						/////////////===== negative result =====///////////////
						resultPixels[row + i] = 255 - pixels[row + i]; //red
						resultPixels[row + i + 1] = 255 - pixels[row + i + 1]; //green
						resultPixels[row + i + 2] = 255 - pixels[row + i + 2]; //blue							
					}else if(this.mode === 'grayScale'){
						///////////===== gray scale result =====///////////////
						let average = (pixels[row + i] + pixels[row + i + 1] + pixels[row + i + 2])/3
						resultPixels[row + i] = average; //red
						resultPixels[row + i + 1] = average; //green
						resultPixels[row + i + 2] = average; //blue	
					}else if(this.mode === 'brigher'){
						///////////===== brighter image =====///////////////
						resultPixels[row + i] = (pixels[row + i] + 50 < 255) ? (pixels[row + i] + 50) : 255; //red
						resultPixels[row + i + 1] = (pixels[row + i + 1] + 50 < 255) ? (pixels[row + i + 1] + 50) : 255; //green
						resultPixels[row + i + 2] = (pixels[row + i + 2] + 50 < 255) ? (pixels[row + i + 2] + 50) : 255; //blue							
					}else if(this.mode === 'darker'){
						///////////===== darker image =====///////////////
						resultPixels[row + i] = (pixels[row + i] - 50 > 0) ? (pixels[row + i] - 50) : 0; //red
						resultPixels[row + i + 1] = (pixels[row + i + 1] - 50 > 0) ? (pixels[row + i + 1] - 50) : 0; //green
						resultPixels[row + i + 2] = (pixels[row + i + 2] - 50 > 0) ? (pixels[row + i + 2] - 50) : 0; //blue							
					}else if(this.mode === 'R&B'){
						///////////===== changing red and blue =====///////////////
						resultPixels[row + i] = pixels[row + i +2]; //red
						resultPixels[row + i + 1] = pixels[row + i + 1]; //green
						resultPixels[row + i + 2] = pixels[row + i]; //blue							
					}
				}
				}
				init += 5;
				}
				console.log("totalRed : " , this.totalRed)
				console.log("totalGreen : " , this.totalGreen)
				console.log("totalBlue : " , this.totalBlue)
				resultContext.putImageData(resultImageData, 0, 0, 0, 0, resultContext.canvas.width,resultContext.canvas.height);
			},
			addRGB(red,green,blue){
				this.totalRed = this.totalRed + red;
				this.totalGreen = this.totalGreen + green;
				this.totalBlue = this.totalBlue + blue;
			},
			changeMode(){
				if(this.cover1src){
					this.changeImage();
				}
			},
			uploadCover1() {
				this.cover1src = "";
				let file = this.$refs.cover1.files;
				if (file && file[0]) {
					let data = new FormData()
					var reader = new FileReader()
					reader.onload = (e) => {
						this.cover1src = e.target.result;
						setTimeout(() => {
							this.changeImage();	
						}, 0);
						
					}
					reader.readAsDataURL(file[0])
					data.append('cover', file[0])
				}
			},
		},

	}
</script>
<style scoped>
	.page-wrapper {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		z-index: 5000;
	}
	.flex-wrapper {
			display: flex;
			flex-direction: row-reverse;
			justify-content: center;
			align-items: center;
			gap: 10px
		}
	/*-------------------- inputs -------------------------------*/

	label {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		margin: 5px 2px;
		width: 100%;
	}

	label span {
		font-size: 13px;
		margin: 0 5px 5px 0;
		color: rgba(0, 0, 0, 0.8);
	}
	/*-------------------- attachments -------------------------------*/
	.attach {
		display: flex;
		flex-direction: column;
		justify-content: center;
    	align-items: center;
	}

	.attach p {
		font-size: 14px;
		color: var(--text-color-secondary);
		margin-bottom: 15px;
	}

	.upload {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		cursor: pointer;
		padding: 10px;
		position: relative;
		z-index: 1;
		margin-top: auto;
	}

	.upload .upload-box {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: relative;
	}

	.upload span {
		font-size: 13px;
		font-weight: 400;
		color: var(--darkGreyText);
		text-align: center;
	}

	.upload img {
		display: none;
	}
	
	.upload-box canvas {
		width: 400px;
		height: 300px;
		border-radius: 4px;
		border: 1px dashed var(--darkGreyText);
	}

	.upload .uploaded-img {
		width: 100% !important;
		height: 110px;
		margin: 0;
		border-radius: 5px;
		object-fit: cover;
		z-index: 1;
	}

	.upload :deep(.progress-circle__container) {
		position: absolute;
		top: calc(50% - 22.5px);
		left: calc(50% - 22.5px);
		z-index: 2;
	}

	.input-file {
		display: none;
		width: 100%;
	}



	/*		Media		*/
	/*////////////////////////////////////////////////////////////////////////////////////////////////////////////////*/

	@media (max-width: 767px) {
		.flex-wrapper {
			flex-wrap: wrap;
		}

		.attach {
			width: 100%;
		}
	}


	@media (max-width: 575px) {
		.card-wrapper {
			width: 100%;
		}
	}
	@media (max-width: 425px) {
		canvas {
			width: 100% !important;
		}
	}

</style>
