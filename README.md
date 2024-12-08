# Training Tesseract-OCR with Custom Data and Font

## Important

These instructions superseed https://github.com/mangerlab/training-tesseract-ocr-with-custom-data-and-font and work with Tesseract 4.0+ which supports LSTM models.

## Prerequisites

1. **Install Tesseract**: Install Tesseract 4.0+ (e.g., `apt install tesseract-ocr tesseract-ocr-dev` or equivalent on Ubuntu).
2. **Prepare Training Data**: Images' sample and their corresponding manual transcriptions. Optionally, get the same fonts used in the images for better results.

## Train a Custom Model

1. **Transcribe a sample of your images**: It is important to transcribe the text as is (e.g., if there is a double space, transcribe it as such). For example, see `megaman-example.png` and `megaman-example.gt.txt`, there is a double space between "game" and "start".
2. **Generate Synthetic Data**: Use the `text2image` tool to create synthetic training data (e.g. `text2image --font='Mega Man X' --text=start.gp.txt --outputbase=start`. This will create tif and box files.
3. **Extract Box Files**:  Run Tesseract to generate `.box` files (e.g., `tesseract start.tif start lstm.train`). This creates an lstmf file, then create a list file with the training file (e.g., `echo 'start.lstmf' | tee -a start.txt`).
4. **Verify and Correct Box Files**: Manually edit the `.box` files if necessary to ensure that character bounding boxes match the actual text content.
5. **Create Unicharset**: Extract character sets from the `.box` files (e.g., `unicharset_extractor start.box`). This creates a `unicharset` file.
6. **Prepare Font Properties**: Create a `font_properties` file listing your fonts (e.g., `echo 'Mega Man X 0 0 0 0 0' | tee -a font_properties`).
7. **Download pre-trained data**: This provides a starting point to train the model (e.g. `wget https://github.com/tesseract-ocr/tessdata_best/raw/main/eng.traineddata`). From this data, create an initial model (e.g., `combine_tessdata -e eng.traineddata eng.lstm`).
8. **Train the model**: Use the new training files to expand the existing pre-trained model (e.g.`lstmtraining --model_output=start_checkpoint --continue_from=eng.lstm --traineddata=eng.traineddata --train_listfile=start.txt --max_iterations=4000; mv start_checkpoint_checkpoint start.lstm`)
9. **Finalize the training checkpoint**: Move the training checkpoint into a finalized model (e.g., `lstmtraining --stop_training --continue_from=start.lstm --traineddata=eng.traineddata --model_output=start.traineddata`).
10. **Check the model**: Apply OCR to the original image to check the accuracy of the trained model (e.g., `export TESSDATA_PREFIX=/home/pacha/github/training-tesseract-ocr-with-custom-data-and-font-2/megaman-example/; tesseract start.tif start --tessdata-dir=. --oem 1 -l start; lstmeval --model=start.traineddata --traineddata=eng.traineddata --eval_listfile=start.txt`).
