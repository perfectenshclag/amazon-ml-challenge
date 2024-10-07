# ML Challenge Solution

Code Documentation: Entity Information Extraction from Images using PaddleOCR
Overview:
This Python code processes images to extract specific entity values such as "item weight", "item volume", "voltage", etc., using Optical Character Recognition (OCR) and regex matching. The images are fetched from URLs provided in a CSV file, preprocessed to improve OCR accuracy, and the extracted information is saved back to a CSV file.
Modules Used:
Pandas (pd): For handling CSV file operations and dataframes.
Requests: To download images from the web using URLs.
Pillow (PIL): To manipulate and preprocess images.
NumPy (np): For numerical operations on images.
OpenCV (cv2): Used for image processing tasks like thresholding and blurring.
PaddleOCR: OCR tool used to extract text from images.
Regex (re): To define patterns for extracting specific entity information.
Logging: To provide detailed debug information throughout the process.

Key Components:
1. Unit Normalization:
A unit_normalization_map standardises unit representations in the text extracted from images. For instance, "kg" is normalised to "kilogram", "lb" to "pound", etc. This ensures consistent units for later processing as only certain units are allowed in the final test_out csv file.
CODE: 
unit_normalization_map = {
    'g': 'gram', 'kg': 'kilogram', 'lbs': 'pound', 'ml': 'milliliter', 
    'v': 'volt', 'w': 'watt', 'cm': 'centimeter', 'm': 'meter'
}

2. Reading the CSV:
The read_csv function from the Pandas reads the input CSV file containing URLs of images and the corresponding entity names. It reads all the image_links and thus downloads and processes the images and thus gets them ready for the PaddleOCR.
CODE:
def read_csv(file_path):
    return pd.read_csv(file_path)

3. Image Preprocessing:
To improve OCR accuracy, images are preprocessed with several steps:
Resize: Enlarge the image to make the text clearer.
Contrast Enhancement: Improves visibility of text.
Binarisation: Converts the image to black-and-white for cleaner text extraction.
Denoising: Reduces noise in the image using Gaussian blur.
Adaptive Thresholding: Applies a more dynamic threshold to improve text visibility.
CODE: 
def preprocess_image(image):
    image = resize_image(image)
    image = enhance_contrast(image)
    image = binarize_image(image)
    image = denoise_image(image)
    image = adaptive_threshold(image)
    return image

4. OCR with PaddleOCR:
The ocr_paddleocr function uses PaddleOCR to extract text from preprocessed images. This function processes each image, extracts the text in blocks, and returns the extracted text in a structured format. It is used for improved accuracy over EasyOCR and Pytesseract.
CODE:
def ocr_paddleocr(image, ocr_instance):
    result = ocr_instance.ocr(np.array(image), cls=True)
    ...
    extracted_text_str = "\n".join(extracted_text)
    return extracted_text_str if extracted_text else ""

5. Regex for Entity Extraction:
Each entity (e.g., weight, volume, voltage) has a corresponding regular expression pattern for extracting numerical values and units. These regular expressions are mainly used to search for the relevant information asked in the “entity_name” and contain the relevant units. For example, the pattern for extracting "item_weight" is defined as follows: 
CODE:
regex_patterns = {
    'item_weight': r'(\d+(?:\.\d+)?)\s*(g|kg|lb?|ounce|gram|pound?)',
    'voltage': r'(\d+(?:\.\d+)?)\s*(v|volt|mv|millivolt)'
}

The extract_info function runs the OCR, applies the regex pattern based on the entity name, and extracts the matched value and unit.
6. Predictions and Saving Results:
The predict_and_save function iterates over the rows in the CSV, fetching images, running OCR, and applying regex to extract relevant information. The extracted data is appended to a list, which is then saved to an output CSV file.
CODE:
def predict_and_save(csv_data, output_file_path, ocr_instance):
    predictions = []
    cache = {}  # Caches OCR results to avoid redundant processing
    for index, row in csv_data.iterrows():
        image_url = row['image_link']
        entity_name = row['entity_name']
        predicted_value = extract_info(image_url, entity_name, ocr_instance, cache)
        predictions.append({'index': index, 'prediction': predicted_value})
    
    predictions_df = pd.DataFrame(predictions)
    predictions_df.to_csv(output_file_path, index=False)


Detailed Walkthrough of Functions:
normalize_unit(unit):
Uses a dictionary to convert shorthand unit representations (like kg, g, etc.) into their full forms (kilogram, gram, etc.).
format_value(value):
Keeps the extracted numeric value in the correct format without modifying it.
Image Preprocessing Functions:
resize_image(): Enlarges the image for better OCR results.
enhance_contrast(): Boosts image contrast.
binarize_image(): Converts the image to black and white.
denoise_image(): Removes noise using Gaussian blur.
adaptive_threshold(): Applies dynamic thresholding to improve text clarity.
OCR Functions:
ocr_paddleocr(): Calls PaddleOCR to extract text from an image.
run_ocr(): Preprocesses the image, performs OCR, and extracts text.
Main Prediction Function:
extract_info(): Fetches the image from a URL, preprocesses it, performs OCR, and uses regex to extract specific entity information (like weight or voltage).

Logging:
Logging is used extensively to provide detailed information on each step of the process. This helps in debugging and tracking the flow of the program, such as loading images, normalizing units, and performing OCR.
CODE:
logging.info("Preprocessing image for OCR")
logging.debug(f"Normalized unit '{unit}' to '{normalized}'")


Main Execution Flow:
Read the CSV file.
Initialize PaddleOCR.
Iterate over each image URL, preprocess the image, run OCR, and extract information using regex.
Save the predictions to a new CSV file.
CODE: 
if __name__ == "__main__":
    main()


Summary:
This Python code reads the CSV file extracts link to the images and then processes images to extract entity values using OCR. It leverages PaddleOCR for text extraction, preprocesses images for better accuracy, and uses regular expressions to identify specific patterns. The results are then stored in a CSV file, making it useful for automating the extraction of structured data from images.

### Appendix

```
entity_unit_map = {
  "width": {
    "centimetre",
    "foot",
    "millimetre",
    "metre",
    "inch",
    "yard"
  },
  "depth": {
    "centimetre",
    "foot",
    "millimetre",
    "metre",
    "inch",
    "yard"
  },
  "height": {
    "centimetre",
    "foot",
    "millimetre",
    "metre",
    "inch",
    "yard"
  },
  "item_weight": {
    "milligram",
    "kilogram",
    "microgram",
    "gram",
    "ounce",
    "ton",
    "pound"
  },
  "maximum_weight_recommendation": {
    "milligram",
    "kilogram",
    "microgram",
    "gram",
    "ounce",
    "ton",
    "pound"
  },
  "voltage": {
    "millivolt",
    "kilovolt",
    "volt"
  },
  "wattage": {
    "kilowatt",
    "watt"
  },
  "item_volume": {
    "cubic foot",
    "microlitre",
    "cup",
    "fluid ounce",
    "centilitre",
    "imperial gallon",
    "pint",
    "decilitre",
    "litre",
    "millilitre",
    "quart",
    "cubic inch",
    "gallon"
  }
}
```
