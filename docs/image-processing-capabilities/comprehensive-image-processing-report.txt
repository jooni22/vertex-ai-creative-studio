# Comprehensive Image Processing Capabilities Report

## Executive Summary

This document provides a comprehensive analysis of image processing, editing, and generation capabilities demonstrated in the `vertex-ai-creative-studio` repository. The analysis covers both AI-powered models (Google's Imagen, Gemini 2.5 Flash Image, Virtual Try-On) and traditional image processing techniques using Python libraries (PIL/Pillow, OpenCV).

## Table of Contents

1. [Google Cloud Image Models](#google-cloud-image-models)
2. [Image Processing with Python Libraries](#image-processing-with-python-libraries)
3. [Mask Generation and Segmentation](#mask-generation-and-segmentation)
4. [LLM-Based Image Capabilities](#llm-based-image-capabilities)
5. [Code Examples and Implementations](#code-examples-and-implementations)
6. [Open-Source Models for Image Processing](#open-source-models-for-image-processing)
7. [References and Additional Resources](#references-and-additional-resources)

---

## 1. Google Cloud Image Models

### 1.1 Imagen Models

The repository extensively uses Google's Imagen family of models for various image generation and editing tasks:

#### Available Imagen Models

| Model Name | Model ID | Purpose | Capabilities |
|------------|----------|---------|--------------|
| **Imagen 2** | `imagegeneration@006` | Legacy image generation | Basic text-to-image |
| **Imagen Nano** | `imagegeneration@004` | Lightweight generation | Fast, lower quality |
| **Imagen 3** | `imagen-3.0-generate-002` | Standard generation | High-quality text-to-image |
| **Imagen 3 Fast** | `imagen-3.0-fast-generate-001` | Fast generation | Faster inference, good quality |
| **Imagen 4** | `imagen-4.0-generate-001` | Latest generation | Enhanced quality and features |
| **Imagen 4 Fast** | `imagen-4.0-fast-generate-001` | Fast latest generation | Optimized speed with quality |
| **Imagen 4 Ultra** | `imagen-4.0-ultra-generate-001` | Premium generation | Highest quality available |
| **Imagen 3 Editing** | `imagen-3.0-capability-001` | Image editing | Inpainting, outpainting, removal, background swap |
| **Imagen Product Recontext** | `imagen-product-recontext-preview-06-30` | Product placement | Place products in new scenes |

**Reference:** `config/default.py` lines 140-150

### 1.2 Imagen Editing Capabilities

Imagen 3 Editing model (`imagen-3.0-capability-001`) provides sophisticated editing features:

#### Edit Modes

1. **Inpaint Insertion** (`EDIT_MODE_INPAINT_INSERTION`)
   - Add new objects or content to specific areas of an image
   - Uses mask-based targeting
   - Supports multiple mask modes (foreground, background, semantic)

2. **Inpaint Removal** (`EDIT_MODE_INPAINT_REMOVAL`)
   - Remove unwanted objects or content from images
   - Automatically fills removed areas with contextually appropriate content

3. **Background Swap** (`EDIT_MODE_BGSWAP`)
   - Replace image backgrounds while preserving foreground subjects
   - Ideal for product photography and recontextualization

4. **Outpainting** (`EDIT_MODE_OUTPAINT`)
   - Extend images beyond their original boundaries
   - Generates contextually appropriate content for expanded areas
   - Used in character consistency workflow (see `models/character_consistency.py`)

5. **Default/Mask-Free Editing** (`EDIT_MODE_DEFAULT`)
   - Make general edits without explicit masks
   - Controlled by text prompts describing desired changes

**Reference:** `experiments/mcp-genmedia/mcp-genmedia-go/plans/IMAGEN_EDITING.md`

#### Mask Modes

The editing API supports multiple mask generation modes:

- **MASK_MODE_FOREGROUND**: Automatically detects and masks foreground objects
- **MASK_MODE_BACKGROUND**: Automatically detects and masks background
- **MASK_MODE_SEMANTIC**: Uses semantic segmentation classes (194 object types available)
- **MASK_MODE_USER_PROVIDED**: User supplies custom mask
- **MASK_MODE_PROMPT** (Descriptive): Uses text description to identify mask area

**Reference:** `components/constants.py` lines 44-88, 147-288

### 1.3 Virtual Try-On (VTO)

A specialized model for fashion and apparel applications:

- **Model ID**: `virtual-try-on-preview-08-04`
- **Purpose**: Virtually place clothing on person images
- **Use Cases**: E-commerce, fashion visualization, outfit planning

#### VTO Capabilities:
- Realistic clothing placement on person images
- Maintains body proportions and poses
- Handles various clothing types and styles
- Concurrent processing for multiple outfit trials

**Implementation:** `models/vto.py`, `experiments/VTO/VTOatScale.ipynb`

**Reference:** `experiments/VTO/README.md`

### 1.4 Gemini 2.5 Flash Image Generation

A newer multimodal approach to image generation:

- **Model ID**: `gemini-2.5-flash-image-preview`
- **Location**: `global`
- **Purpose**: Image generation with conversational context
- **Key Features**:
  - Can generate images from text prompts
  - Can use reference images to guide generation
  - Supports iterative refinement through conversation
  - Integrated with Gemini's language understanding

**Implementation:** `models/gemini.py`, `pages/gemini_image_generation.py`

**Capabilities:**
- Multi-image input support
- Reference-guided generation
- Transformation suggestions (AI-generated edit prompts)
- Image critique and analysis
- Sequential generation chains (track related images)

**Reference:** `config/default.py` lines 66-71, `pages/gemini_image_generation.py`

---

## 2. Image Processing with Python Libraries

### 2.1 PIL/Pillow Usage

The repository extensively uses PIL (Python Imaging Library) for non-AI image processing tasks.

#### Core PIL Operations Found in Codebase:

**File:** `models/character_consistency.py`

1. **Image Loading and Format Conversion**
   ```python
   from PIL import Image as PIL_Image
   pil_image = PIL_Image.open(io.BytesIO(image_bytes))
   ```

2. **Image Dimension Analysis**
   ```python
   width, height = pil_image.size
   aspect_ratio = "9:16" if height > width else "16:9"
   ```

3. **Mask Creation**
   ```python
   # Create blank mask
   mask = PIL_Image.new("L", initial_image.size, 0)
   ```

4. **Image Padding and Resizing**
   ```python
   # Thumbnail resizing (maintains aspect ratio)
   image_pil.thumbnail(target_size)
   
   # Create padded canvas
   source_image_padded = PIL_Image.new(mode, target_size, color=(fill_val, fill_val, fill_val))
   source_image_padded.paste(source_image, (insert_pt_x, insert_pt_y))
   ```

5. **Image Format Conversion**
   ```python
   def _get_bytes_from_pil(image: PIL_Image.Image) -> bytes:
       """Gets the image bytes from a PIL Image object."""
       byte_io_png = io.BytesIO()
       image.save(byte_io_png, "PNG")
       return byte_io_png.getvalue()
   ```

6. **Outpainting Preparation**
   - Calculate target dimensions (16:9 aspect ratio)
   - Pad images to larger canvas
   - Create corresponding masks for padded areas
   - Align masks with padded images

**Reference:** `models/character_consistency.py` lines 28, 324-326, 380-450

### 2.2 OpenCV (cv2) Usage

OpenCV is used for video processing and frame extraction:

**File:** `models/video_processing.py`

#### OpenCV Capabilities Demonstrated:

1. **Video Frame Extraction**
   ```python
   import cv2
   # Used for reading video files and extracting frames
   ```

2. **Video Processing Integration**
   - Works with MoviePy for video editing
   - Frame-by-frame processing
   - Video composition and transitions

3. **Frame Extraction for Veo3 Item Consistency**
   ```python
   # experiments/veo3-item-consistency/extend_video/extract_frame.py
   def extract_last_frames(video_path: str, num_frames: int = 4) -> list:
       """Extracts the last 'num_frames' from a video file using OpenCV."""
   ```

**Reference:** `models/video_processing.py` line 19, `experiments/veo3-item-consistency/extend_video/extract_frame.py`

### 2.3 Additional Image Processing Libraries

The repository also utilizes:

1. **NumPy** - Array operations for image data manipulation
   ```python
   import numpy as np
   # Used for efficient array operations on image data
   ```

2. **SciPy** - Advanced image transformations
   ```python
   from scipy.ndimage import gaussian_filter, map_coordinates
   from scipy.special import expit
   ```

3. **scikit-image** - Image resizing and transformations
   ```python
   from skimage.transform import resize
   ```

4. **MoviePy** - Video processing and composition
   ```python
   from moviepy import VideoFileClip, afx, vfx
   ```

**Reference:** `models/video_processing.py` lines 19-27

---

## 3. Mask Generation and Segmentation

### 3.1 Automatic Mask Generation

The Imagen editing API provides sophisticated automatic mask generation:

#### Semantic Segmentation Classes

The API supports **194 distinct object classes** for automatic segmentation, including:

**Common Objects:**
- People (125), Animals (6-16), Furniture (57-92)
- Vehicles (175-185), Food items (45-56)
- Electronics (37-42), Household items

**Complete List Available In:** `components/constants.py` lines 147-288

#### Mask Generation Modes

1. **Foreground Mode**
   - Automatically identifies and masks primary subjects
   - Useful for subject-focused editing

2. **Background Mode**
   - Automatically identifies and masks everything except main subjects
   - Ideal for background replacement

3. **Semantic Mode**
   - Uses object class IDs for precise targeting
   - Example: Class ID 8 = "dog", Class ID 85 = "mirror"
   - Allows multi-object selection via class ID array

4. **Descriptive/Prompt Mode**
   - Uses natural language descriptions to identify mask areas
   - Leverages AI to understand spatial and semantic descriptions

### 3.2 Mask Dilation

All mask modes support **mask dilation** parameter:
- Range: 0.0 to 1.0 (percentage of mask expansion)
- Allows fine-tuning of mask boundaries
- Useful for capturing edge details or creating smooth transitions

### 3.3 Manual Mask Creation (PIL)

The codebase demonstrates manual mask creation for custom workflows:

```python
# Create blank grayscale mask
mask = PIL_Image.new("L", initial_image.size, 0)

# Mask is then padded to match outpainting requirements
mask_pil_padded = _pad_to_target_size(mask_pil, target_size, ...)
```

**Use Case:** Outpainting workflow in character consistency
**Reference:** `models/character_consistency.py` lines 383-405

### 3.4 Mask-Free Editing

Some editing operations don't require explicit masks:
- **Controlled Editing Mode**: AI interprets text instructions
- **Style Transfer**: Applied globally or to semantically understood regions
- **General Modifications**: Color grading, lighting adjustments

**Reference:** `experiments/mcp-genmedia/mcp-genmedia-go/plans/IMAGEN_EDITING.md` lines 189-209

---

## 4. LLM-Based Image Capabilities

### 4.1 Gemini 2.5 Flash Image Generation

Gemini's multimodal capabilities enable sophisticated image workflows:

#### Core Capabilities:

1. **Text-to-Image Generation**
   - Natural language prompts
   - Contextual understanding from conversation
   - Multi-turn refinement

2. **Reference-Guided Generation**
   ```python
   generated_images, execution_time = generate_image_from_prompt_and_images(
       prompt=final_prompt,
       images=input_gcs_uris,  # Reference images
       gcs_folder="gemini_image_generations",
       file_prefix="gemini_image",
   )
   ```

3. **Image Critique and Analysis**
   ```python
   critique_text = image_critique(generation_instruction, image_output)
   ```
   - Analyzes generated images
   - Provides feedback on quality, composition, adherence to prompt

4. **Transformation Prompt Generation**
   ```python
   raw_transformations = generate_transformation_prompts(image_uris=[gcs_uri])
   ```
   - AI suggests potential edits/transformations
   - Generates creative variation prompts
   - Provides title and detailed prompt for each suggestion

**Implementation:** `pages/gemini_image_generation.py`, `models/gemini.py`

### 4.2 Character Consistency Workflow

A sophisticated multi-step LLM-orchestrated workflow:

#### Workflow Steps:

1. **Facial Analysis** (Gemini)
   - Extract detailed facial features as structured data
   - Create composite facial profile from multiple reference images

2. **Natural Language Description** (Gemini)
   - Convert structured facial data to natural language
   - Generate prompts suitable for image generation models

3. **Scene Prompt Generation** (Gemini)
   - Combine character description with scene requirements
   - Generate both positive and negative prompts
   - Temperature: 0.3 (balanced creativity)

4. **Candidate Generation** (Parallel)
   - Imagen: Generate images with reference image guidance
   - Gemini: Generate alternative images

5. **Best Image Selection** (Gemini)
   - Analyze all candidates
   - Select best match based on character consistency and scene quality
   - Temperature: 0.2 (analytical)

6. **Outpainting** (Imagen)
   - Expand selected image to desired aspect ratio (16:9)
   - Use PIL for padding and mask creation

7. **Video Generation** (Veo)
   - Generate cinematic prompt with Gemini
   - Create video from outpainted image

**Reference:** `models/character_consistency.py`, `plans/character_consistency_plan.md`

### 4.3 Prompt Engineering for Image Models

The repository demonstrates advanced prompt engineering:

1. **Structured Prompts**
   ```python
   full_prompt = f"{input_txt}, {prompt_modifiers_segment}"
   ```

2. **Negative Prompts**
   - Filter unwanted elements
   - Improve output quality
   - Model-specific optimization

3. **Reference Image Integration**
   - Subject consistency
   - Style transfer
   - Composition guidance

4. **Few-Shot Customization**
   - Multiple reference images
   - Control images (face mesh, pose)
   - Subject descriptions

**Reference:** `models/image_models.py`, `experiments/mcp-genmedia/mcp-genmedia-go/plans/IMAGEN_EDITING.md`

---

## 5. Code Examples and Implementations

### 5.1 Image Generation (Imagen)

**File:** `models/image_models.py`

```python
def generate_images(
    model: str,
    prompt: str,
    number_of_images: int,
    aspect_ratio: str,
    negative_prompt: str,
):
    """Imagen image generation with Google GenAI client"""
    client = ImagenModelSetup.init(model_id=model)
    cfg = Default()
    
    gcs_output_directory = f"gs://{cfg.IMAGE_BUCKET}/{cfg.IMAGEN_GENERATED_SUBFOLDER}"
    
    response = client.models.generate_images(
        model=model,
        prompt=prompt,
        config=types.GenerateImagesConfig(
            number_of_images=number_of_images,
            include_rai_reason=True,
            output_gcs_uri=gcs_output_directory,
            aspect_ratio=aspect_ratio,
            negative_prompt=negative_prompt,
        ),
    )
    return response
```

**Features:**
- Automatic retry logic with exponential backoff
- GCS output path configuration
- RAI (Responsible AI) filtering
- Multiple aspect ratios support

### 5.2 Image Editing (Imagen)

**File:** `models/image_models.py`

```python
def edit_image(
    model: str,
    prompt: str,
    edit_mode: str,
    mask_mode: str,
    reference_image_bytes: bytes,
    number_of_images: int,
):
    """Edits an image using the Google GenAI client."""
    client = ImagenModelSetup.init(model_id=model)
    cfg = Default()
    gcs_output_directory = f"gs://{cfg.IMAGE_BUCKET}/{cfg.IMAGEN_EDITED_SUBFOLDER}"
    
    raw_ref_image = types.RawReferenceImage(
        reference_id=1,
        reference_image=reference_image_bytes,
    )
    
    mask_ref_image = types.MaskReferenceImage(
        reference_id=2,
        config=types.MaskReferenceConfig(
            mask_mode=mask_mode,
            mask_dilation=0,
        ),
    )
    
    response = client.models.edit_image(
        model=model,
        prompt=prompt,
        reference_images=[raw_ref_image, mask_ref_image],
        config=types.EditImageConfig(
            edit_mode=edit_mode,
            number_of_images=number_of_images,
            include_rai_reason=True,
            output_gcs_uri=gcs_output_directory,
            output_mime_type="image/jpeg",
        ),
    )
    
    return edited_uris
```

**Key Points:**
- Dual reference images (raw + mask)
- Configurable mask modes
- Multiple edit modes
- GCS output storage

### 5.3 Outpainting with PIL

**File:** `models/character_consistency.py`

```python
def _outpaint_image(image_bytes: bytes, prompt: str) -> bytes:
    """Performs outpainting on an image to a 16:9 aspect ratio."""
    client = genai.Client(vertexai=True, project=cfg.PROJECT_ID, location=cfg.LOCATION)
    edit_model = cfg.CHARACTER_CONSISTENCY_IMAGEN_MODEL
    
    # Load image and create mask
    initial_image = PIL_Image.open(io.BytesIO(image_bytes))
    mask = PIL_Image.new("L", initial_image.size, 0)
    
    # Calculate target dimensions
    target_height = 1080
    target_width = int(target_height * 16 / 9)
    target_size = (target_width, target_height)
    
    # Pad image and mask to target size
    image_pil_outpaint, mask_pil_outpaint = _pad_image_and_mask(
        initial_image,
        mask,
        target_size,
        vertical_offset_ratio=0.5,
        horizontal_offset_ratio=0.5,
    )
    
    # Convert to bytes for API
    image_for_api = types.Image(image_bytes=_get_bytes_from_pil(image_pil_outpaint))
    mask_for_api = types.Image(image_bytes=_get_bytes_from_pil(mask_pil_outpaint))
    
    # Create reference images
    raw_ref_image = types.RawReferenceImage(
        reference_id=1, reference_image=image_for_api
    )
    mask_ref_image = types.MaskReferenceImage(
        reference_image=mask_for_api,
        config=types.MaskReferenceConfig(
            mask_mode="MASK_MODE_USER_PROVIDED",
            mask_dilation=0.03,
        ),
    )
    
    # Perform outpainting
    edited_image_response = client.models.edit_image(
        model=edit_model,
        prompt=prompt,
        reference_images=[raw_ref_image, mask_ref_image],
        config=types.EditImageConfig(
            edit_mode="EDIT_MODE_OUTPAINT",
            number_of_images=1,
        ),
    )
    
    return edited_image_response.generated_images[0].image.image_bytes
```

**Demonstrates:**
- PIL image loading and manipulation
- Mask creation from scratch
- Image padding to target aspect ratio
- Integration with Imagen editing API

### 5.4 Virtual Try-On

**File:** `experiments/VTO/VTOatScale.ipynb` (key functions)

```python
def run_tryon(person_b64, name, b64):
    """Runs VTO prediction for a single outfit."""
    start = time.time()
    client = PredictionServiceClient(
        client_options={"api_endpoint": f"{LOCATION}-aiplatform.googleapis.com"}
    )
    instances = [{
        "personImage": {"image": {"bytesBase64Encoded": person_b64}},
        "productImages": [{"image": {"bytesBase64Encoded": b64}}],
    }]
    response = client.predict(
        endpoint=model_endpoint, 
        instances=instances, 
        parameters={}
    )
    elapsed = time.time() - start
    output_img = prediction_to_pil_image(response.predictions[0])
    return output_img, elapsed
```

**Features:**
- Base64 encoding for API transfer
- Concurrent processing with ThreadPoolExecutor
- PIL for image resizing and display
- Time tracking for performance analysis

### 5.5 Product Recontextualization

**File:** `models/image_models.py`

```python
def recontextualize_product_in_scene(
    image_uris_list: list[str], prompt: str, sample_count: int
) -> list[str]:
    """Recontextualizes a product in a scene and returns a list of GCS URIs."""
    cfg = Default()
    client_options = {"api_endpoint": f"{cfg.LOCATION}-aiplatform.googleapis.com"}
    client = aiplatform.gapic.PredictionServiceClient(client_options=client_options)
    
    model_endpoint = f"projects/{cfg.PROJECT_ID}/locations/{cfg.LOCATION}/publishers/google/models/{cfg.MODEL_IMAGEN_PRODUCT_RECONTEXT}"
    
    instance = {"productImages": []}
    for product_image_uri in image_uris_list:
        product_image = {"image": {"gcsUri": product_image_uri}}
        instance["productImages"].append(product_image)
    
    if prompt:
        instance["prompt"] = prompt
    
    parameters = {"sampleCount": sample_count}
    
    response = client.predict(
        endpoint=model_endpoint, instances=[instance], parameters=parameters
    )
    
    # Decode and store results
    gcs_uris = []
    for prediction in response.predictions:
        if prediction.get("bytesBase64Encoded"):
            encoded_mask_string = prediction["bytesBase64Encoded"]
            mask_bytes = base64.b64decode(encoded_mask_string)
            gcs_uri = store_to_gcs(
                folder="recontext_results",
                file_name=f"recontext_result_{uuid.uuid4()}.png",
                mime_type="image/png",
                contents=mask_bytes,
                decode=False,
            )
            gcs_uris.append(gcs_uri)
    
    return gcs_uris
```

**Purpose:** Place product images in new scenes/contexts
**Use Cases:** E-commerce, marketing, product visualization

---

## 6. Open-Source Models for Image Processing

While the repository primarily uses Google's proprietary models, it's worth noting compatible open-source alternatives for various image processing tasks:

### 6.1 Image Generation Models

1. **Stable Diffusion**
   - Text-to-image generation
   - Inpainting and outpainting
   - ControlNet for guided generation
   - Available via Hugging Face: `stabilityai/stable-diffusion-xl-base-1.0`

2. **DALL-E Mini / Craiyon**
   - Lightweight text-to-image
   - Open-source alternative to DALL-E

3. **Midjourney Alternatives**
   - DreamStudio (Stability AI)
   - FLUX (Black Forest Labs)

### 6.2 Image Segmentation Models

1. **Segment Anything Model (SAM)**
   - Facebook/Meta's powerful segmentation model
   - Zero-shot segmentation
   - Interactive mask generation
   - Repository: `facebookresearch/segment-anything`

2. **DeepLab v3+**
   - Semantic segmentation
   - Pre-trained on COCO, Pascal VOC
   - Available in TensorFlow and PyTorch

3. **Mask R-CNN**
   - Instance segmentation
   - Object detection with masks
   - Implementation: `matterport/Mask_RCNN`

### 6.3 Image Editing Models

1. **InstructPix2Pix**
   - Edit images with text instructions
   - No mask required
   - Repository: `timothybrooks/instruct-pix2pix`

2. **ControlNet**
   - Fine-grained control over image generation
   - Supports edge detection, pose, depth maps
   - Works with Stable Diffusion

3. **GFPGAN / Real-ESRGAN**
   - Face restoration
   - Image super-resolution
   - Practical restoration tools

### 6.4 Image-to-Image Translation

1. **Pix2Pix / CycleGAN**
   - Style transfer
   - Domain adaptation
   - Unpaired image-to-image translation

2. **StyleGAN**
   - High-quality face generation
   - Style manipulation
   - NVIDIA's state-of-the-art GAN

### 6.5 Traditional CV Libraries

These are already used in the repository:

1. **OpenCV (cv2)**
   - Comprehensive computer vision library
   - Image filtering, transformations, feature detection
   - Video processing

2. **PIL/Pillow**
   - Basic image operations
   - Format conversion
   - Drawing and text overlay

3. **scikit-image**
   - Image processing algorithms
   - Morphological operations
   - Filtering and feature extraction

4. **NumPy**
   - Array-based image manipulation
   - Mathematical operations on pixels

### 6.6 Face and Person-Related Models

1. **MediaPipe**
   - Google's ML solutions for face/body detection
   - Face mesh, pose estimation
   - Hand tracking

2. **FaceNet / ArcFace**
   - Face recognition and verification
   - Feature extraction

3. **OpenPose**
   - Body pose estimation
   - Multi-person detection

### 6.7 Model Deployment Platforms

For deploying open-source models:

1. **Hugging Face**
   - Model hub with thousands of pre-trained models
   - Easy deployment with Inference API

2. **Replicate**
   - Run open-source models via API
   - No infrastructure management

3. **Modal**
   - Serverless deployment for ML models
   - GPU support

### 6.8 Comparison: Google vs. Open-Source

| Feature | Google Models | Open-Source |
|---------|--------------|-------------|
| **Quality** | State-of-the-art, consistent | Varies, but many SOTA options |
| **Ease of Use** | Managed API, no setup | Requires setup, deployment |
| **Cost** | Pay-per-use pricing | Infrastructure costs |
| **Customization** | Limited, API-driven | Full control, fine-tuning possible |
| **Privacy** | Data sent to Google | Can run locally |
| **Support** | Enterprise support available | Community-driven |
| **Updates** | Automatic | Manual updates required |

---

## 7. References and Additional Resources

### 7.1 Documentation in Repository

1. **Main README**: `/README.md`
   - Overview of GenMedia Creative Studio
   - Deployment instructions
   - Feature list

2. **Imagen Product Recontextualization**
   - `/experiments/Imagen_Product_Recontext/README.md`
   - Notebooks: `imagen_product_recontext_at_scale.ipynb`

3. **Virtual Try-On**
   - `/experiments/VTO/README.md`
   - Notebook: `VTOatScale.ipynb`

4. **MCP Imagen Documentation**
   - `/experiments/mcp-genmedia/mcp-genmedia-go/mcp-imagen-go/README.md`
   - `/experiments/mcp-genmedia/mcp-genmedia-go/plans/IMAGEN_EDITING.md`

5. **Character Consistency**
   - `/experiments/veo3-character-consistency/README.md`
   - Workflow notebook: `workflow.ipynb`

6. **Configuration**
   - `/config/default.py` - Model IDs and settings
   - `/components/constants.py` - Edit modes, mask modes, semantic classes

### 7.2 Google Cloud Documentation

1. **Vertex AI Imagen API**
   - https://cloud.google.com/vertex-ai/generative-ai/docs/image/overview

2. **Imagen 3 Image Generation**
   - https://cloud.google.com/vertex-ai/generative-ai/docs/image/generate-images

3. **Imagen 3 Image Editing**
   - https://cloud.google.com/vertex-ai/generative-ai/docs/image/edit-images

4. **Virtual Try-On**
   - https://cloud.google.com/vertex-ai/generative-ai/docs/image/virtual-try-on

5. **Gemini Multimodal**
   - https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/overview

### 7.3 Key Code Files

**Image Generation and Editing:**
- `/models/image_models.py` - Core Imagen API wrappers
- `/models/gemini.py` - Gemini image generation functions
- `/pages/edit_images.py` - Image editing UI
- `/pages/imagen.py` - Imagen generation UI
- `/pages/gemini_image_generation.py` - Gemini image UI

**Image Processing:**
- `/models/character_consistency.py` - PIL-based image manipulation
- `/models/video_processing.py` - OpenCV and MoviePy usage
- `/models/vto.py` - Virtual Try-On implementation

**Configuration:**
- `/config/default.py` - Model configurations
- `/components/constants.py` - UI constants and options

**Experiments:**
- `/experiments/VTO/VTOatScale.ipynb`
- `/experiments/Imagen_Product_Recontext/imagen_product_recontext_at_scale.ipynb`
- `/experiments/veo3-character-consistency/workflow.ipynb`

### 7.4 External Resources

**Python Libraries:**
1. **Pillow**: https://pillow.readthedocs.io/
2. **OpenCV**: https://docs.opencv.org/
3. **NumPy**: https://numpy.org/doc/
4. **scikit-image**: https://scikit-image.org/docs/
5. **MoviePy**: https://zulko.github.io/moviepy/

**Open-Source Models:**
1. **Hugging Face**: https://huggingface.co/
2. **Segment Anything**: https://github.com/facebookresearch/segment-anything
3. **Stable Diffusion**: https://github.com/Stability-AI/stablediffusion
4. **ControlNet**: https://github.com/lllyasviel/ControlNet

---

## Appendix A: Semantic Segmentation Class IDs

The Imagen editing API supports 194 semantic segmentation classes. Here's a subset of the most common:

| Class ID | Object | Class ID | Object | Class ID | Object |
|----------|--------|----------|--------|----------|--------|
| 0 | backpack | 8 | dog | 37 | laptop |
| 1 | umbrella | 9 | horse | 85 | mirror |
| 2 | bag | 10 | sheep | 125 | person |
| 6 | bird | 25 | washer dryer | 175 | bicycle |
| 7 | cat | 28 | toilet | 176 | car |

**Full list available in:** `/components/constants.py` lines 147-288

---

## Appendix B: Aspect Ratios Supported

All Imagen models support these aspect ratios:
- 1:1 (square)
- 3:4 (portrait)
- 4:3 (landscape)
- 9:16 (mobile portrait)
- 16:9 (widescreen)

**Reference:** `/components/constants.py` lines 24-30, 134-140

---

## Appendix C: Image Processing Workflows

### Workflow 1: Character Consistency (7 Steps)
1. Download reference images
2. Generate facial descriptions (Gemini)
3. Generate scene prompt (Gemini)
4. Generate candidate images (Imagen + Gemini)
5. Select best image (Gemini)
6. Outpaint to 16:9 (Imagen + PIL)
7. Generate video (Veo)

### Workflow 2: Product Recontextualization
1. Upload product images
2. Specify scene description
3. Generate recontextualized images (Imagen Product Recontext)
4. Evaluate results (optional, using Gemini)

### Workflow 3: Virtual Try-On at Scale
1. Upload person image
2. Load multiple product/outfit images
3. Concurrent VTO predictions
4. Display side-by-side results

---

## Conclusion

The `vertex-ai-creative-studio` repository demonstrates comprehensive image processing capabilities spanning:

1. **State-of-the-art AI models** from Google (Imagen, Gemini, VTO)
2. **Traditional image processing** with PIL/Pillow and OpenCV
3. **Advanced mask generation** with semantic segmentation (194 classes)
4. **LLM-orchestrated workflows** for complex multi-step tasks
5. **Integration patterns** for combining AI and traditional methods

The codebase serves as both a production application and an educational resource, showing practical implementations of:
- Text-to-image generation
- Image editing (inpainting, outpainting, removal, background swap)
- Mask-based and mask-free editing
- Virtual try-on for fashion
- Product recontextualization
- Character/subject consistency across generations
- Video generation from images

For developers looking to build similar capabilities, this repository provides:
- **Working code examples** for all major features
- **Configuration patterns** for model selection and parameters
- **Error handling** and retry logic for production use
- **Integration patterns** for GCS storage and Firestore metadata
- **UI implementations** using Mesop framework

The combination of Google's proprietary models with open-source alternatives provides flexibility for various use cases, budgets, and deployment requirements.

---

**Document Version:** 1.0  
**Last Updated:** 2025-01-22  
**Repository:** https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio
