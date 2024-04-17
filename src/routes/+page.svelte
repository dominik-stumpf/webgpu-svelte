<script lang="ts">
	import triangleVertWGSL from "./triangle.vert.wgsl?raw";
	import redFragWGSL from "./red.frag.wgsl?raw";
	import { browser } from "$app/environment";

	async function main() {
		const canvas = document.querySelector(
			"canvas",
		) as HTMLCanvasElement;
		const adapter = await navigator.gpu.requestAdapter();
		if (adapter === null) {
			throw new Error("no gpu adapter found");
		}
		const device = await adapter.requestDevice();

		const context = canvas.getContext("webgpu") as GPUCanvasContext;

		const devicePixelRatio = window.devicePixelRatio;
		canvas.width = canvas.clientWidth * devicePixelRatio;
		canvas.height = canvas.clientHeight * devicePixelRatio;
		const presentationFormat =
			navigator.gpu.getPreferredCanvasFormat();

		context.configure({
			device,
			format: presentationFormat,
			alphaMode: "premultiplied",
		});

		const pipeline = device.createRenderPipeline({
			layout: "auto",
			vertex: {
				module: device.createShaderModule({
					code: triangleVertWGSL,
				}),
			},
			fragment: {
				module: device.createShaderModule({
					code: redFragWGSL,
				}),
				targets: [
					{
						format: presentationFormat,
					},
				],
			},
			primitive: {
				topology: "triangle-list",
			},
		});

		function frame() {
			const commandEncoder = device.createCommandEncoder();
			const textureView = context
				.getCurrentTexture()
				.createView();

			const renderPassDescriptor: GPURenderPassDescriptor = {
				colorAttachments: [
					{
						view: textureView,
						clearValue: [0, 0, 0, 1],
						loadOp: "clear",
						storeOp: "store",
					},
				],
			};

			const passEncoder =
				commandEncoder.beginRenderPass(
					renderPassDescriptor,
				);
			passEncoder.setPipeline(pipeline);
			passEncoder.draw(3);
			passEncoder.end();

			device.queue.submit([commandEncoder.finish()]);
			requestAnimationFrame(frame);
		}

		requestAnimationFrame(frame);
	}

	if (browser) {
		main();
	}
</script>

<canvas />

<style>
	canvas {
		height: 100vh;
		width: 100vw;
		display: block;
	}
	:global(body) {
		background: black;
		margin: 0;
	}
</style>
