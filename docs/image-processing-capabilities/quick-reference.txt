# Image Processing Quick Reference Guide

Quick lookup for common image processing tasks, model IDs, and code patterns.

## Model IDs

### Imagen Models

```python
# Image Generation
MODEL_IMAGEN_3 = "imagen-3.0-generate-002"
MODEL_IMAGEN_3_FAST = "imagen-3.0-fast-generate-001"
MODEL_IMAGEN_4 = "imagen-4.0-generate-001"
MODEL_IMAGEN_4_FAST = "imagen-4.0-fast-generate-001"
MODEL_IMAGEN_4_ULTRA = "imagen-4.0-ultra-generate-001"

# Image Editing
MODEL_IMAGEN_EDITING = "imagen-3.0-capability-001"

# Specialized
MODEL_IMAGEN_PRODUCT_RECONTEXT = "imagen-product-recontext-preview-06-30"
MODEL_VTO = "virtual-try-on-preview-08-04"
```

### Gemini Models

```python
# Multimodal Image Generation
MODEL_GEMINI_IMAGE = "gemini-2.5-flash-image-preview"

# General Purpose (with vision)
MODEL_GEMINI_FLASH = "gemini-2.5-flash"
```

## Edit Modes

```python
# Inpainting - Add Content
edit_mode = "EDIT_MODE_INPAINT_INSERTION"

# Inpainting - Remove Content
edit_mode = "EDIT_MODE_INPAINT_REMOVAL"

# Background Replacement
edit_mode = "EDIT_MODE_BGSWAP"

# Extend Image Boundaries
edit_mode = "EDIT_MODE_OUTPAINT"

# General Editing (no mask)
edit_mode = "EDIT_MODE_DEFAULT"
```

## Mask Modes

```python
# Automatic Foreground Detection
mask_mode = "MASK_MODE_FOREGROUND"

# Automatic Background Detection
mask_mode = "MASK_MODE_BACKGROUND"

# Semantic Segmentation (specify class IDs)
mask_mode = "MASK_MODE_SEMANTIC"
segmentation_classes = [8]  # dog

# Descriptive (text-based)
mask_mode = "MASK_MODE_PROMPT"

# Custom Mask Image
mask_mode = "MASK_MODE_USER_PROVIDED"
```

## Aspect Ratios

```python
aspect_ratio = "1:1"    # Square
aspect_ratio = "16:9"   # Widescreen
aspect_ratio = "9:16"   # Portrait/Mobile
aspect_ratio = "4:3"    # Standard
aspect_ratio = "3:4"    # Portrait Standard
```

## Common Semantic Classes

| Class ID | Object | Class ID | Object | Class ID | Object |
|----------|--------|----------|--------|----------|--------|
| 6 | bird | 37 | laptop | 125 | person |
| 7 | cat | 42 | television | 175 | bicycle |
| 8 | dog | 66 | bed | 176 | car |
| 9 | horse | 67 | table | 178 | motorcycle |
| 28 | toilet | 85 | mirror | 179 | airplane |

[Full list of 194 classes in main documentation]

## Code Snippets

### Generate Image (Imagen)

```python
from models.image_models import generate_images
from config.default import Default

cfg = Default()

response = generate_images(
    model=cfg.MODEL_IMAGEN_3_FAST,
    prompt="a beautiful sunset over mountains",
    number_of_images=1,
    aspect_ratio="16:9",
    negative_prompt="people, text, watermark"
)

# Access generated images
for img in response.generated_images:
    gcs_uri = img.image.gcs_uri
    image_bytes = img.image.image_bytes
```

### Edit Image (Inpaint - Add Object)

```python
from models.image_models import edit_image
from config.default import Default

cfg = Default()

edited_uris = edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="a red sports car",
    edit_mode="EDIT_MODE_INPAINT_INSERTION",
    mask_mode="MASK_MODE_FOREGROUND",
    reference_image_bytes=original_image_bytes,
    number_of_images=1
)
```

### Remove Object (Semantic Mask)

```python
edited_uris = edit_image(
    model="imagen-3.0-capability-001",
    prompt="",  # Empty for removal
    edit_mode="EDIT_MODE_INPAINT_REMOVAL",
    mask_mode="MASK_MODE_SEMANTIC",
    reference_image_bytes=image_bytes,
    number_of_images=1
)

# Need to pass segmentation_classes separately via MaskReferenceImage
# See full implementation in models/image_models.py
```

### Replace Background

```python
edited_uris = edit_image(
    model="imagen-3.0-capability-001",
    prompt="a modern minimalist studio with white walls and soft lighting",
    edit_mode="EDIT_MODE_BGSWAP",
    mask_mode="MASK_MODE_BACKGROUND",
    reference_image_bytes=product_image_bytes,
    number_of_images=1
)
```

### Outpaint Image

```python
# Requires padding the image first with PIL
from PIL import Image
import io

# Load and pad image
original = Image.open(io.BytesIO(image_bytes))
# ... padding logic (see character_consistency.py)

edited_uris = edit_image(
    model="imagen-3.0-capability-001",
    prompt="continue the scene naturally",
    edit_mode="EDIT_MODE_OUTPAINT",
    mask_mode="MASK_MODE_USER_PROVIDED",
    reference_image_bytes=padded_image_bytes,
    number_of_images=1
)
```

### Generate with Gemini

```python
from models.gemini import generate_image_from_prompt_and_images

gcs_uris, execution_time = generate_image_from_prompt_and_images(
    prompt="a futuristic cityscape at night with neon lights",
    images=[],  # Optional reference images
    gcs_folder="gemini_generations",
    file_prefix="city"
)
```

### Generate with Reference Images

```python
gcs_uris, execution_time = generate_image_from_prompt_and_images(
    prompt="Create a similar scene but in winter with snow",
    images=["gs://bucket/reference-image.png"],
    gcs_folder="gemini_generations",
    file_prefix="winter_scene"
)
```

### Virtual Try-On

```python
from google.cloud import aiplatform
from google.cloud.aiplatform.gapic import PredictionServiceClient

client = PredictionServiceClient(
    client_options={"api_endpoint": f"{location}-aiplatform.googleapis.com"}
)

model_endpoint = f"projects/{project_id}/locations/{location}/publishers/google/models/virtual-try-on-preview-08-04"

instances = [{
    "personImage": {"image": {"bytesBase64Encoded": person_b64}},
    "productImages": [{"image": {"bytesBase64Encoded": outfit_b64}}],
}]

response = client.predict(
    endpoint=model_endpoint,
    instances=instances,
    parameters={}
)
```

## PIL/Pillow Common Operations

### Load Image

```python
from PIL import Image
import io

# From bytes
pil_image = Image.open(io.BytesIO(image_bytes))

# From file
pil_image = Image.open("path/to/image.jpg")

# From URL (with requests)
import requests
response = requests.get(image_url)
pil_image = Image.open(io.BytesIO(response.content))
```

### Get Image Info

```python
width, height = pil_image.size
mode = pil_image.mode  # 'RGB', 'RGBA', 'L', etc.
format = pil_image.format  # 'JPEG', 'PNG', etc.
```

### Resize Image

```python
# Resize to exact dimensions
new_image = pil_image.resize((800, 600))

# Resize maintaining aspect ratio (thumbnail)
pil_image.thumbnail((800, 600))  # Modifies in-place
```

### Create New Image

```python
# RGB image with white background
new_image = Image.new("RGB", (800, 600), color=(255, 255, 255))

# Grayscale image (for masks)
mask = Image.new("L", (800, 600), 0)  # Black mask
```

### Crop Image

```python
# Define crop box (left, top, right, bottom)
box = (100, 100, 400, 400)
cropped = pil_image.crop(box)
```

### Paste Image

```python
# Paste small_image onto canvas at position (x, y)
canvas.paste(small_image, (100, 100))

# With mask for transparency
canvas.paste(small_image, (100, 100), mask=mask)
```

### Convert to Bytes

```python
# PNG
byte_io = io.BytesIO()
pil_image.save(byte_io, format="PNG")
image_bytes = byte_io.getvalue()

# JPEG with quality
byte_io = io.BytesIO()
pil_image.save(byte_io, format="JPEG", quality=90)
image_bytes = byte_io.getvalue()
```

### Convert Color Mode

```python
# Convert to RGB
rgb_image = pil_image.convert("RGB")

# Convert to grayscale
gray_image = pil_image.convert("L")

# Add alpha channel
rgba_image = pil_image.convert("RGBA")
```

## OpenCV Operations

### Read Video

```python
import cv2

cap = cv2.VideoCapture("video.mp4")
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    # Process frame
cap.release()
```

### Extract Frame

```python
cap = cv2.VideoCapture("video.mp4")
cap.set(cv2.CAP_PROP_POS_FRAMES, frame_number)
ret, frame = cap.read()
cap.release()
```

### Convert Frame to PIL

```python
import cv2
from PIL import Image

# OpenCV uses BGR, PIL uses RGB
frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
pil_image = Image.fromarray(frame_rgb)
```

## Storage Operations

### Upload to GCS

```python
from common.storage import store_to_gcs

gcs_uri = store_to_gcs(
    folder="my_folder",
    file_name="image.png",
    mime_type="image/png",
    contents=image_bytes
)
# Returns: "gs://bucket/my_folder/image.png"
```

### Download from GCS

```python
from common.storage import download_from_gcs

image_bytes = download_from_gcs("gs://bucket/path/to/image.png")
```

### Convert GCS URI to HTTPS URL

```python
from common.utils import gcs_uri_to_https_url

https_url = gcs_uri_to_https_url("gs://bucket/path/to/image.png")
# Returns: "https://storage.googleapis.com/bucket/path/to/image.png"
```

### Convert HTTPS URL to GCS URI

```python
from common.utils import https_url_to_gcs_uri

gcs_uri = https_url_to_gcs_uri("https://storage.googleapis.com/bucket/path/to/image.png")
# Returns: "gs://bucket/path/to/image.png"
```

## Safety and Content Filtering

### Safety Filter Levels

```python
# Imagen generation/editing
safety_filter_level = "BLOCK_LOW_AND_ABOVE"    # Most restrictive
safety_filter_level = "BLOCK_MEDIUM_AND_ABOVE"  # Balanced
safety_filter_level = "BLOCK_ONLY_HIGH"         # Permissive
safety_filter_level = "BLOCK_NONE"              # No filtering
```

### Person Generation Settings

```python
# Imagen generation/editing
person_generation = "DONT_ALLOW"   # No people
person_generation = "ALLOW_ADULT"  # Only adults
person_generation = "ALLOW_ALL"    # All ages
```

## Error Handling

### Retry Logic

```python
from tenacity import (
    retry,
    retry_if_exception_type,
    stop_after_attempt,
    wait_exponential,
)

@retry(
    wait=wait_exponential(multiplier=1, min=1, max=10),
    stop=stop_after_attempt(3),
    retry=retry_if_exception_type(Exception),
    reraise=True,
)
def generate_with_retry():
    return generate_images(...)
```

### Check for Generation Errors

```python
response = generate_images(...)

if response.generated_images:
    for img in response.generated_images:
        if hasattr(img, 'error') and img.error:
            print(f"Generation error: {img.error}")
        elif hasattr(img, 'image') and img.image:
            # Success
            gcs_uri = img.image.gcs_uri
else:
    print("No images generated")
```

## Best Practices

### Prompting

1. **Be specific**: Include details about style, lighting, composition
2. **Use negative prompts**: Exclude unwanted elements
3. **Set aspect ratio**: Match your use case (16:9 for web, 9:16 for mobile)
4. **Iterate**: Use reference images and refine prompts

### Image Processing

1. **Always validate dimensions**: Check image size before processing
2. **Handle aspect ratios**: Use thumbnail() to maintain proportions
3. **Use appropriate formats**: PNG for transparency, JPEG for photos
4. **Optimize quality**: Balance file size and visual quality
5. **Error handling**: Wrap PIL operations in try-except blocks

### API Usage

1. **Implement retry logic**: Network failures happen
2. **Use appropriate models**: Fast models for prototyping, standard for production
3. **Batch operations**: Process multiple images concurrently when possible
4. **Monitor costs**: Track API usage and optimize
5. **Cache results**: Store generated images in GCS

### Security

1. **Validate inputs**: Check file types and sizes
2. **Use safety filters**: Appropriate for your use case
3. **Sanitize prompts**: Remove potentially harmful instructions
4. **Rate limiting**: Implement on client side
5. **Access control**: Use IAP and proper GCS permissions

## Performance Tips

### Image Generation

- Use "fast" models for iteration (`imagen-3.0-fast-generate-001`)
- Generate multiple images in one call when possible
- Use lower resolution for drafts, higher for final
- Cache common generations

### Image Processing

- Resize images before processing when possible
- Use thumbnail() instead of resize() to maintain aspect ratio
- Process images in parallel with ThreadPoolExecutor
- Use appropriate JPEG quality (80-90 is usually sufficient)

### Storage

- Store generated images in GCS immediately
- Use appropriate bucket locations (same region as Vertex AI)
- Implement lifecycle policies for temporary images
- Use signed URLs for secure access

## Common Issues and Solutions

### Issue: "Image too large for API"

```python
from PIL import Image

max_dimension = 4096
if width > max_dimension or height > max_dimension:
    pil_image.thumbnail((max_dimension, max_dimension))
```

### Issue: "Mask doesn't match image size"

```python
# Ensure mask has same dimensions as image
mask = Image.new("L", pil_image.size, 0)
```

### Issue: "RGBA to RGB conversion for JPEG"

```python
if pil_image.mode == "RGBA":
    # Create white background
    background = Image.new("RGB", pil_image.size, (255, 255, 255))
    background.paste(pil_image, mask=pil_image.split()[3])  # Use alpha as mask
    pil_image = background
```

### Issue: "Out of memory with large images"

```python
# Resize before processing
pil_image.thumbnail((2048, 2048))

# Or process in chunks
# (implementation depends on specific use case)
```

---

## Related Documentation

- [Comprehensive Report](./comprehensive-image-processing-report.md) - Full analysis
- [Use Case Examples](./use-case-examples.md) - Practical scenarios
- [Main README](./README.md) - Overview and index

## Repository Files

Key files for reference:
- `models/image_models.py` - Core implementations
- `config/default.py` - Model IDs and configuration
- `components/constants.py` - UI constants and options
- `common/storage.py` - GCS operations
- `common/utils.py` - Utility functions

---

**Last Updated:** 2025-01-22
