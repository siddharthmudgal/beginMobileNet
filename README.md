# Training a mobile net model and deploying on an android device

websites that helped : 
https://codelabs.developers.google.com/codelabs/tensorflow-for-poets


1. Clone the tensorflow-for-poets-2 repository

	git clone https://github.com/googlecodelabs/tensorflow-for-poets-2
	cd tensorflow-for-poets-2
2. Get sample database if you dont have one
	curl http://download.tensorflow.org/example_images/flower_photos.tgz \
    | tar xz -C tf_files

    otherwise copy your dataset into tf_files inside a dir, where each subdir represents a class of objects

3. export these variables : 

	IMAGE_SIZE=224
	ARCHITECTURE="mobilenet_0.50_${IMAGE_SIZE}"

4. Run the retrain script from inside tensorflow-for-poets-2 dir
	
	python -m scripts.retrain \
  --bottleneck_dir=tf_files/bottlenecks \
  --how_many_training_steps=500 \
  --model_dir=tf_files/models/ \
  --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}" \
  --output_graph=tf_files/retrained_graph.pb \
  --output_labels=tf_files/retrained_labels.txt \
  --architecture="${ARCHITECTURE}" \
  --image_dir=tf_files/flower_photos

  #playing more with training steps
  python -m scripts.retrain \
  --bottleneck_dir=tf_files/bottlenecks \
  --model_dir=tf_files/models/"${ARCHITECTURE}" \
  --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}" \
  --output_graph=tf_files/retrained_graph.pb \
  --output_labels=tf_files/retrained_labels.txt \
  --architecture="${ARCHITECTURE}" \
  --image_dir=tf_files/flower_photos

5. testing the model 
	python -m scripts.label_image \
    --graph=tf_files/retrained_graph.pb  \
    --image=tf_files/flower_photos/daisy/21652746_cc379e0eea_m.jpg

6. help for retrain script :

	python -m scripts.retrain -h


## optimize the models
	
1. python -m tensorflow.python.tools.optimize_for_inference \
  --input=tf_files/retrained_graph.pb \
  --output=tf_files/optimized_graph.pb \
  --input_names="input" \
  --output_names="final_result"
2. verify the optimized model 
	
	python -m scripts.label_image \
  --graph=tf_files/retrained_graph.pb\
  --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg

  python -m scripts.label_image \
    --graph=tf_files/optimized_graph.pb \
    --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg

3. Quantize model and compress
	
	python -m scripts.quantize_graph \
  --input=tf_files/optimized_graph.pb \
  --output=tf_files/rounded_graph.pb \
  --output_node_names=final_result \
  --mode=weights_rounded

  gzip -c tf_files/rounded_graph.pb > tf_files/rounded_graph.pb.gz

  gzip -l tf_files/rounded_graph.pb.gz

4. evaluate rounded model

	python -m scripts.evaluate  tf_files/optimized_graph.pb

	python -m scripts.evaluate  tf_files/rounded_graph.pb

5. Copy the rounded_graph and labels.txt file to assets folder in the android project included with android studio

6. Change output name in Classifier activity 

	private static final String INPUT_NAME = "input";
  	private static final String OUTPUT_NAME = "final_result";

  	Also, check variables 


	  private static final String MODEL_FILE = "file:///android_asset/rounded_graph.pb";
	  private static final String LABEL_FILE = "file:///android_asset/retrained_labels.txt";


