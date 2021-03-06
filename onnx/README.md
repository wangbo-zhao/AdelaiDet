
# export to onnx
example: python onnx/export_model_to_onnx.py --config-file configs/FCOS-Detection/R_50_1x.yaml --opts MODEL.WEIGHTS weights/fcos_R_50_1x.pth

# norm in FCOS Head
The norm in FOCS head is GroupNorm(GN) by default. Unlike BN, GN caculates the mean and var of the features online. Thus, it costs extra time and memory.
On the other sisde, as BN can be fused into previous convolution layer, we can regard BN having no cost in inference. The following instruction introduces a simple method to measure the impact of GN on speed.

1. prepare certain amount of images (for example 1000) in folder output/test/input/

2. add time measurement code in demo/demo.py

3. GN + GPU: total execution time 285.1398s, average 0.0696s per image

python demo/demo --config-file configs/FCOS-Detection/R_50_1x.yaml --input output/test/input/ --output output/test/output/  --opts MODEL.WEIGHTS weights/fcos_R_50_1x.pth

2. BN + GPU: total execution time 257.4333s, average 0.0628s per image

python demo/demo.py --config-file configs/FCOS-Detection/R_50_1x.yaml --input output/test/input/ --output output/test/output/  --opts MODEL.WEIGHTS weights/fcos_R_50_1x.pth MODEL.FCOS.NORM BN

3. GN + CPU: total execution time 1125.4375s, average 1.0112s per image

python demo/demo.py --config-file configs/FCOS-Detection/R_50_1x.yaml --input output/test/input/ --output output/test/output/  --opts MODEL.WEIGHTS weights/fcos_R_50_1x.pth MODEL.DEVICE cpu

4. BN + CPU: total execution time 1068.0550s, average 0.9596s per image

python demo/demo.py --config-file configs/FCOS-Detection/R_50_1x.yaml --input output/test/input/ --output output/test/output/  --opts MODEL.WEIGHTS weights/fcos_R_50_1x.pth MODEL.DEVICE cpu MODEL.FCOS.NORM BN

Speedup test on 2080ti. The result shows 5~10% slow of GN compared with BN.
