<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const isDragging = ref(false)
const selectedFile = ref<File | null>(null)

const scale = ref(1)
const offsetX = ref(0)
const offsetY = ref(0)
const showSuccess = ref(false)
const isMobile = ref(false)
const MOBILE_BREAKPOINT = 1024
const showError = ref(false)

function updateScale() {
	const vw = window.innerWidth
	const vh = window.innerHeight
	isMobile.value = vw < MOBILE_BREAKPOINT
	if (isMobile.value) {
		scale.value = 1
		offsetX.value = 0
		offsetY.value = 0
		return
	}
	const baseW = 1920
	const baseH = 1080
	const s = Math.min(vw / baseW, vh / baseH)
	scale.value = s
	offsetX.value = Math.max(0, (vw - baseW * s) / 2)
	offsetY.value = Math.max(0, (vh - baseH * s) / 2)
}

onMounted(() => {
	updateScale()
	window.addEventListener('resize', updateScale)
})

onUnmounted(() => {
	window.removeEventListener('resize', updateScale)
})

function onDragOver(event: DragEvent) {
	event.preventDefault()
	isDragging.value = true
}

function onDragLeave(event: DragEvent) {
	event.preventDefault()
	isDragging.value = false
}

function onDrop(event: DragEvent) {
	event.preventDefault()
	isDragging.value = false
	const file = event.dataTransfer?.files?.[0]
	if (file) selectedFile.value = file
}

function onPickFile(event: Event) {
	const input = event.target as HTMLInputElement
	const file = input.files?.[0]
	if (file) selectedFile.value = file
}

function triggerPick() {
	const input = document.getElementById('resume-input') as HTMLInputElement
	input?.click()
}

function onFinish() {
	if (!selectedFile.value) {
		showError.value = true
		setTimeout(() => {
			showError.value = false
		}, 1500)
		return
	}
	showSuccess.value = true
	selectedFile.value = null
	setTimeout(() => {
		showSuccess.value = false
	}, 1500)
}
</script>

<template>
	<div class="upload-resume">
		<!-- Desktop: pixel-perfect stage scaled to viewport -->
		<div v-if="!isMobile" class="viewport">
			<div
				class="stage"
				:style="{ transform: `translate(${offsetX}px, ${offsetY}px) scale(${scale})` }"
			>
				<div class="illustration" />
				<div class="panel" />

				<h1 class="title">finish!<br />Upload your resume</h1>
				<p class="subtitle">
					Upload your resume, the platform will help you parse and optimize, you can also skip this step
				</p>

				<p class="label-upload">Upload file</p>

				<div
					class="dropzone"
					:class="{ dragging: isDragging }"
					@dragover="onDragOver"
					@dragleave="onDragLeave"
					@drop="onDrop"
					@click="triggerPick"
				>
					<input id="resume-input" type="file" accept=".pdf,.doc,.docx" @change="onPickFile" />
					<div class="upload-icon" />
					<p class="hint">
						Drag your resume file to this area, or click on the area to select the appropriate file to upload
					</p>
					<p v-if="selectedFile" class="file-name">Selected: {{ selectedFile?.name }}</p>
				</div>

				<button class="btn btn-light btn-last" type="button">Last step</button>
				<button class="btn btn-primary btn-finish" type="button" @click="onFinish">finish</button>

				<div class="progress">
					<div class="bar full" />
					<div class="bar active" />
					<div class="steps">
						<div class="step done"><span class="check" /></div>
						<div class="step done"><span class="check" /></div>
						<div class="step current">3</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Mobile: stacked vertical layout -->
		<div v-else class="mobile">
			<div class="m-illustration" />
			<div class="m-card">
				<h1 class="m-title">finish!<br />Upload your resume</h1>
				<p class="m-subtitle">
					Upload your resume, the platform will help you parse and optimize, you can also skip this step
				</p>
				<p class="m-label">Upload file</p>

				<div
					class="m-dropzone"
					:class="{ dragging: isDragging }"
					@dragover="onDragOver"
					@dragleave="onDragLeave"
					@drop="onDrop"
					@click="triggerPick"
				>
					<input id="resume-input" type="file" accept=".pdf,.doc,.docx" @change="onPickFile" />
					<div class="upload-icon" />
					<p class="hint">
						Drag your resume file here or tap to select a file to upload
					</p>
					<p v-if="selectedFile" class="file-name">Selected: {{ selectedFile?.name }}</p>
				</div>

				<div class="m-actions">
					<button class="btn btn-light" type="button">Last step</button>
					<button class="btn btn-primary" type="button" @click="onFinish">finish</button>
				</div>

				<div class="m-progress">
					<div class="bar full" />
					<div class="bar active" />
					<div class="steps">
						<div class="step done"><span class="check" /></div>
						<div class="step done"><span class="check" /></div>
						<div class="step current">3</div>
					</div>
				</div>
			</div>
		</div>

		<div v-if="showSuccess" class="modal">
			<div class="modal-content">
				<div class="success-icon" />
				<div class="modal-title">上传成功</div>
			</div>
		</div>
		<div v-if="showError" class="modal">
			<div class="modal-content">
				<div class="error-icon" />
				<div class="modal-title">请先上传！</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
.upload-resume {
	position: relative;
	width: 100%;
	height: 100%;
	background: #f8fbf9;
	overflow: hidden;
}

/* Desktop viewport (applies to desktop only) */
.viewport {
	position: absolute;
	left: 0;
	top: 0;
	right: 0;
	bottom: 0;
	overflow: hidden;
}

.stage {
	position: absolute;
	left: 0;
	top: 0;
	width: 1920px;
	height: 1080px;
	transform-origin: top left;
}

.illustration {
	position: absolute;
	left: 0;
	top: 511px;
	width: 763px;
	height: 569px;
	background: url('../assets/figma/social-illustration-7bffb9.png') no-repeat 0 0/cover;
	pointer-events: none;
}

.panel {
	position: absolute;
	left: 875px;
	top: 103px;
	width: 943px;
	height: 922px;
	background: #ffffff;
	box-shadow: 0px 2px 20px rgba(19, 67, 112, 0.15);
	border-radius: 32px;
}

.title {
	position: absolute;
	left: 102px;
	top: 124px;
	margin: 0;
	font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
	font-weight: 400;
	font-size: 40px;
	line-height: 1.375em;
	color: #000000;
	text-align: left;
}

.subtitle {
	position: absolute;
	left: 102px;
	top: 254px;
	width: 696px;
	margin: 0;
	font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
	font-weight: 400;
	font-size: 20px;
	line-height: 1.75em;
	color: #a5a5a5;
	text-align: left;
}

.label-upload {
	position: absolute;
	left: 1004px;
	top: 133px;
	margin: 0;
	font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
	font-weight: 500;
	font-size: 36px;
	letter-spacing: 0.08em;
	color: #060326;
}

.dropzone {
	position: absolute;
	left: 1004px;
	top: 221px;
	width: 685px;
	height: 420px;
	background: #f2faff;
	border-radius: 15px;
	border: 1px dashed #0538bb;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 16px;
	cursor: pointer;
	transition: border-color 0.2s ease, box-shadow 0.2s ease;
}
.dropzone.dragging,
.dropzone:hover {
	outline: 2px dashed #1849d6;
	border-color: #1849d6;
	box-shadow: 0 0 0 4px rgba(24, 73, 214, 0.08) inset;
}
.dropzone input[type='file'] {
	display: none;
}
.upload-icon {
	width: 58px;
	height: 39px;
	background: #1849d6;
	mask: url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"58\" height=\"39\" viewBox=\"0 0 58 39\"><path fill=\"%23fff\" d=\"M33 24v-8h6l-10-10-10 10h6v8h8Zm-22 6V6h9.172l4-4H11a5 5 0 0 0-5 5v24a5 5 0 0 0 5 5h36a5 5 0 0 0 5-5V11.828l-4 4V30H11Z\"/></svg>') center/contain no-repeat;
}
.hint {
	width: 499px;
	text-align: center;
	font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
	font-size: 16px;
	line-height: 2em;
	color: #a5a5a5;
	margin: 0;
}
.file-name {
	margin: 0;
	font-size: 14px;
	color: #1849d6;
}

.btn {
	position: absolute;
	width: 272px;
	height: 67px;
	border-radius: 8px;
	border: none;
	font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
	font-size: 24px;
	letter-spacing: 0.08em;
	cursor: pointer;
	color: #ffffff;
	transition: transform 0.06s ease, filter 0.12s ease;
}
.btn:hover {
	filter: brightness(1.05);
}
.btn:active {
	transform: translateY(1px);
	filter: brightness(0.98);
}
.btn-light {
	background: #75acff;
}
.btn-primary {
	background: #1b5aff;
}
.btn-last {
	left: 1060px;
	top: 739px;
}
.btn-finish {
	left: 1361px;
	top: 739px;
}

.progress {
	position: absolute;
	left: 1175px;
	top: 921px;
	width: 343px;
	height: 32px;
}
.bar.full {
	position: absolute;
	left: 0;
	top: 14px;
	width: 343px;
	height: 4px;
	background: #dde0e7;
	border-radius: 2px;
	z-index: 1;
	pointer-events: none;
}
.bar.active {
	position: absolute;
	left: 0;
	top: 14px;
	width: 299px;
	height: 4px;
	background: #85a7ff;
	border-radius: 2px;
	z-index: 1;
	pointer-events: none;
}
.steps {
	position: absolute;
	left: 0;
	top: 0;
	display: grid;
	grid-template-columns: 32px 32px 32px;
	gap: 96px;
	z-index: 2;
}
.step {
	width: 32px;
	height: 32px;
	border-radius: 16px;
	background: #2a4cfe;
	box-shadow: 0px 7px 64px rgba(0, 0, 0, 0.07);
	display: grid;
	place-items: center;
	color: #ffffff;
	font-weight: 500;
	font-size: 11px;
	line-height: 1.18em;
}
.step.done {
	background: #85a7ff;
	color: transparent;
}
.check {
	width: 10px;
	height: 7px;
	background: #2a4cfe;
	mask: url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"10\" height=\"8\" viewBox=\"0 0 10 8\"><path fill=\"%23000\" d=\"M3.5 6.1 1.4 4 0 5.4 3.5 9 10 2.5 8.6 1.1 3.5 6.1Z\"/></svg>') center/contain no-repeat;
}
.step.current {
	background: #2a4cfe;
}

/* Mobile stacked layout */
.mobile {
	position: absolute;
	inset: 0;
	display: flex;
	flex-direction: column;
	align-items: stretch;
	gap: 16px;
	padding: 16px;
	overflow: auto;
	-webkit-overflow-scrolling: touch;
	overflow-x: hidden;
}
.m-illustration {
	height: 210px;
	background: url('../assets/figma/social-illustration-7bffb9.png') no-repeat center/cover;
	border-radius: 12px;
}
.m-card {
	background: #ffffff;
	border-radius: 16px;
	box-shadow: 0px 2px 20px rgba(19, 67, 112, 0.15);
	padding: 16px;
	display: flex;
	flex-direction: column;
	gap: 12px;
}
.m-title {
	margin: 0;
	font-size: 24px;
	line-height: 1.3;
	color: #000000;
}
.m-subtitle {
	margin: 0;
	font-size: 14px;
	line-height: 1.6;
	color: #a5a5a5;
}
.m-label {
	margin: 4px 0 0 0;
	font-size: 16px;
	font-weight: 600;
	letter-spacing: 0.04em;
	color: #060326;
}
.m-dropzone {
	margin-top: 8px;
	width: 100%;
	height: 220px;
	background: #f2faff;
	border-radius: 12px;
	border: 1px dashed #0538bb;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 12px;
	cursor: pointer;
	transition: border-color 0.2s ease, box-shadow 0.2s ease;
}
.m-dropzone input[type='file'] {
	display: none;
}
.m-dropzone.dragging,
.m-dropzone:hover {
	outline: 2px dashed #1849d6;
	border-color: #1849d6;
	box-shadow: 0 0 0 4px rgba(24, 73, 214, 0.08) inset;
}
.m-actions {
	display: grid;
	grid-template-columns: 1fr 1fr;
	gap: 12px;
	margin-top: 8px;
}
.m-actions .btn {
	position: static;
	height: 52px;
	font-size: 18px;
	width: 100%;
}
.m-progress {
	position: relative;
	margin-top: 8px;
	height: 32px;
	width: 100%;
}
.mobile .m-progress .bar.full {
	width: 100%;
}
.mobile .m-progress .bar.active {
	width: 90%;
}
.mobile .m-progress .steps {
	position: relative;
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-top: 0;
	z-index: 2;
}
.mobile .hint {
	width: auto;
	max-width: 100%;
	padding: 0 16px;
	word-break: break-word;
	overflow-wrap: break-word;
}

/* Success/Error Modal */
.modal {
	position: fixed;
	inset: 0;
	display: grid;
	place-items: center;
	background: rgba(0, 0, 0, 0.35);
	z-index: 1000;
}
.modal-content {
	background: #ffffff;
	border-radius: 16px;
	box-shadow: 0px 7px 64px rgba(0, 0, 0, 0.07);
	padding: 24px 32px;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 12px;
	min-width: 240px;
}
.success-icon {
	width: 48px;
	height: 48px;
	background: #2a4cfe;
	mask: url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 24 24\"><path fill=\"%23fff\" d=\"M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20Zm-1.2 13.2-3.5-3.5 1.4-1.4 2.1 2.1 4.9-4.9 1.4 1.4-6.3 6.3Z\"/></svg>') center/contain no-repeat;
	border-radius: 50%;
}
.error-icon {
	width: 48px;
	height: 48px;
	background: #ff4d4f;
	mask: url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 24 24\"><path fill=\"%23fff\" d=\"M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20Zm-1 5h2v7h-2V7Zm0 9h2v2h-2v-2Z\"/></svg>') center/contain no-repeat;
	border-radius: 50%;
}
.modal-title {
	font-size: 18px;
	color: #213547;
}
</style> 