# Image Processing Use Case Examples

Practical examples for common image processing scenarios using the vertex-ai-creative-studio codebase.

## Table of Contents

1. [Background Removal and Replacement](#1-background-removal-and-replacement)
2. [Object Insertion](#2-object-insertion)
3. [Object Removal](#3-object-removal)
4. [Image Expansion (Outpainting)](#4-image-expansion-outpainting)
5. [Product Photography](#5-product-photography)
6. [Fashion Try-On](#6-fashion-try-on)
7. [Character Consistency](#7-character-consistency)
8. [Image Transformations](#8-image-transformations)
9. [Batch Processing](#9-batch-processing)
10. [Image Composition](#10-image-composition)

---

## 1. Background Removal and Replacement

### Use Case: E-commerce Product Photography

Replace a product's background while keeping the product intact.

#### Scenario A: Simple Background Replacement

```python
from models.image_models import edit_image
from config.default import Default
from common.storage import download_from_gcs, store_to_gcs

cfg = Default()

# Download product image
product_image_bytes = download_from_gcs("gs://my-bucket/products/shoe.jpg")

# Replace background with studio setting
edited_uris = edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="white seamless backdrop with soft professional lighting, studio photography",
    edit_mode="EDIT_MODE_BGSWAP",
    mask_mode="MASK_MODE_BACKGROUND",
    reference_image_bytes=product_image_bytes,
    number_of_images=1
)

print(f"New image: {edited_uris[0]}")
```

#### Scenario B: Multiple Background Variations

```python
backgrounds = [
    "white minimalist studio backdrop",
    "natural wood table with soft shadows",
    "marble countertop with elegant lighting",
    "outdoor garden setting with blurred background"
]

results = []
for bg_prompt in backgrounds:
    edited_uris = edit_image(
        model=cfg.MODEL_IMAGEN_EDITING,
        prompt=bg_prompt,
        edit_mode="EDIT_MODE_BGSWAP",
        mask_mode="MASK_MODE_BACKGROUND",
        reference_image_bytes=product_image_bytes,
        number_of_images=1
    )
    results.append(edited_uris[0])

# Now you have 4 variations of the same product
```

#### Scenario C: Remove Background Completely

```python
# Remove background, leaving only the product
edited_uris = edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="",  # Empty prompt for simple removal
    edit_mode="EDIT_MODE_INPAINT_REMOVAL",
    mask_mode="MASK_MODE_BACKGROUND",
    reference_image_bytes=product_image_bytes,
    number_of_images=1
)
```

---

## 2. Object Insertion

### Use Case: Interior Design Visualization

Add furniture or decor items to room photos.

#### Scenario A: Add Specific Item (Foreground Mask)

```python
# Add a plant to the foreground of a room photo
edited_uris = edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="a tall potted monstera plant in a modern white ceramic pot",
    edit_mode="EDIT_MODE_INPAINT_INSERTION",
    mask_mode="MASK_MODE_FOREGROUND",
    reference_image_bytes=room_image_bytes,
    number_of_images=1
)
```

#### Scenario B: Add Item to Specific Location (Semantic Mask)

```python
# Add artwork above the couch
# First, identify the couch using semantic segmentation
raw_ref_image = types.RawReferenceImage(
    reference_id=1,
    reference_image=room_image_bytes,
)

mask_ref_image = types.MaskReferenceImage(
    reference_id=2,
    reference_image=None,  # Auto-generate
    config=types.MaskReferenceConfig(
        mask_mode="MASK_MODE_SEMANTIC",
        segmentation_classes=[62],  # Class ID 62 = couch
        mask_dilation=0.15,  # Expand mask slightly above couch
    ),
)

response = client.models.edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="a large abstract painting with blue and gold colors in a gold frame",
    reference_images=[raw_ref_image, mask_ref_image],
    config=types.EditImageConfig(
        edit_mode="EDIT_MODE_INPAINT_INSERTION",
        number_of_images=1,
    ),
)
```

#### Scenario C: Add Multiple Items

```python
# Add items one at a time, using previous result as input
items_to_add = [
    ("a modern floor lamp in the corner", "MASK_MODE_FOREGROUND"),
    ("decorative throw pillows on the couch", "MASK_MODE_SEMANTIC"),
    ("a coffee table book and vase", "MASK_MODE_FOREGROUND"),
]

current_image = original_room_bytes

for item_prompt, mask_mode in items_to_add:
    edited_uris = edit_image(
        model=cfg.MODEL_IMAGEN_EDITING,
        prompt=item_prompt,
        edit_mode="EDIT_MODE_INPAINT_INSERTION",
        mask_mode=mask_mode,
        reference_image_bytes=current_image,
        number_of_images=1
    )
    # Download for next iteration
    current_image = download_from_gcs(edited_uris[0])
```

---

## 3. Object Removal

### Use Case: Photo Cleanup

Remove unwanted objects, people, or elements from photos.

#### Scenario A: Remove Specific Object by Semantic Class

```python
# Remove a person from a landscape photo
# Class ID 125 = person

raw_ref_image = types.RawReferenceImage(
    reference_id=1,
    reference_image=photo_bytes,
)

mask_ref_image = types.MaskReferenceImage(
    reference_id=2,
    reference_image=None,
    config=types.MaskReferenceConfig(
        mask_mode="MASK_MODE_SEMANTIC",
        segmentation_classes=[125],  # person
        mask_dilation=0.05,
    ),
)

response = client.models.edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="",  # Empty for removal
    reference_images=[raw_ref_image, mask_ref_image],
    config=types.EditImageConfig(
        edit_mode="EDIT_MODE_INPAINT_REMOVAL",
        number_of_images=1,
    ),
)
```

#### Scenario B: Remove Multiple Objects

```python
# Remove both car and bicycle from street scene
mask_ref_image = types.MaskReferenceImage(
    reference_id=2,
    reference_image=None,
    config=types.MaskReferenceConfig(
        mask_mode="MASK_MODE_SEMANTIC",
        segmentation_classes=[175, 176],  # bicycle, car
        mask_dilation=0.05,
    ),
)

response = client.models.edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="",
    reference_images=[raw_ref_image, mask_ref_image],
    config=types.EditImageConfig(
        edit_mode="EDIT_MODE_INPAINT_REMOVAL",
        number_of_images=1,
    ),
)
```

#### Scenario C: Remove with Descriptive Prompt

```python
# Remove objects using natural language description
edited_uris = edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="",
    edit_mode="EDIT_MODE_INPAINT_REMOVAL",
    mask_mode="MASK_MODE_PROMPT",  # Descriptive mode
    reference_image_bytes=photo_bytes,
    number_of_images=1
)
# Note: Would need to provide mask description via full API
```

---

## 4. Image Expansion (Outpainting)

### Use Case: Extend Images for Different Aspect Ratios

Convert portrait images to landscape or expand canvases.

#### Scenario A: Portrait to Landscape (Character Consistency Example)

```python
from PIL import Image as PIL_Image
import io
from models.character_consistency import _pad_image_and_mask, _get_bytes_from_pil

# Load portrait image
initial_image = PIL_Image.open(io.BytesIO(portrait_bytes))

# Create blank mask (all black = areas to generate)
mask = PIL_Image.new("L", initial_image.size, 0)

# Target 16:9 landscape
target_height = 1080
target_width = int(target_height * 16 / 9)  # 1920
target_size = (target_width, target_height)

# Pad image and mask (centers the original)
image_padded, mask_padded = _pad_image_and_mask(
    initial_image,
    mask,
    target_size,
    vertical_offset_ratio=0.5,
    horizontal_offset_ratio=0.5,
)

# Convert to bytes
image_bytes_padded = _get_bytes_from_pil(image_padded)
mask_bytes_padded = _get_bytes_from_pil(mask_padded)

# Prepare for API
raw_ref_image = types.RawReferenceImage(
    reference_id=1,
    reference_image=types.Image(image_bytes=image_bytes_padded)
)

mask_ref_image = types.MaskReferenceImage(
    reference_image=types.Image(image_bytes=mask_bytes_padded),
    config=types.MaskReferenceConfig(
        mask_mode="MASK_MODE_USER_PROVIDED",
        mask_dilation=0.03,
    ),
)

# Outpaint
response = client.models.edit_image(
    model=cfg.MODEL_IMAGEN_EDITING,
    prompt="continue the scene naturally, maintaining style and lighting",
    reference_images=[raw_ref_image, mask_ref_image],
    config=types.EditImageConfig(
        edit_mode="EDIT_MODE_OUTPAINT",
        number_of_images=1,
    ),
)
```

#### Scenario B: Extend Canvas Vertically

```python
# Add space at top and bottom
original = PIL_Image.open(io.BytesIO(image_bytes))
width, height = original.size

# Add 30% more height
new_height = int(height * 1.3)
extra_height = new_height - height

# Create padded canvas
padded = PIL_Image.new("RGB", (width, new_height), color=(128, 128, 128))
padded.paste(original, (0, extra_height // 2))

# Create mask (mark padded areas)
mask = PIL_Image.new("L", (width, new_height), 255)  # White = generate
mask_draw = PIL_Image.new("L", (width, height), 0)  # Black = keep
mask.paste(mask_draw, (0, extra_height // 2))

# Proceed with outpainting...
```

---

## 5. Product Photography

### Use Case: Generate Professional Product Shots

Place products in various contexts for marketing.

#### Scenario A: Product Recontextualization

```python
from models.image_models import recontextualize_product_in_scene

product_uris = ["gs://my-bucket/products/watch.png"]

scenes = [
    "a luxury watch displayed on a mahogany desk in a modern executive office",
    "a watch on a wrist against a backdrop of mountain adventure",
    "elegant watch placed on velvet cushion in jewelry store display",
]

for scene_prompt in scenes:
    result_uris = recontextualize_product_in_scene(
        image_uris_list=product_uris,
        prompt=scene_prompt,
        sample_count=2  # Generate 2 variations
    )
    print(f"Generated {len(result_uris)} images for: {scene_prompt}")
```

#### Scenario B: Generate Product from Description

```python
from models.image_models import generate_images

# Generate product mockup
response = generate_images(
    model=cfg.MODEL_IMAGEN_4,
    prompt="professional product photography of a minimalist white ceramic coffee mug, side view, on white background, studio lighting, high resolution, commercial photography",
    number_of_images=4,
    aspect_ratio="1:1",
    negative_prompt="shadows, reflections, people, text, low quality"
)
```

#### Scenario C: Lifestyle Product Photography

```python
# Generate product in use
response = generate_images(
    model=cfg.MODEL_IMAGEN_3,
    prompt="overhead shot of a laptop on a wooden desk with a coffee mug, notebook, and succulent plant, warm natural lighting, minimal aesthetic, lifestyle photography",
    number_of_images=1,
    aspect_ratio="16:9",
    negative_prompt="people, text, clutter, dark lighting"
)
```

---

## 6. Fashion Try-On

### Use Case: Virtual Wardrobe Visualization

Try on different outfits virtually.

#### Scenario A: Single Outfit Try-On

```python
from google.cloud.aiplatform.gapic import PredictionServiceClient
import base64

# Encode images
person_b64 = base64.b64encode(person_image_bytes).decode()
outfit_b64 = base64.b64encode(outfit_image_bytes).decode()

# Setup client
client = PredictionServiceClient(
    client_options={"api_endpoint": f"{cfg.LOCATION}-aiplatform.googleapis.com"}
)

model_endpoint = f"projects/{cfg.PROJECT_ID}/locations/{cfg.LOCATION}/publishers/google/models/{cfg.VTO_MODEL_ID}"

# Create request
instances = [{
    "personImage": {"image": {"bytesBase64Encoded": person_b64}},
    "productImages": [{"image": {"bytesBase64Encoded": outfit_b64}}],
}]

# Predict
response = client.predict(
    endpoint=model_endpoint,
    instances=instances,
    parameters={}
)

# Get result
result_b64 = response.predictions[0]["bytesBase64Encoded"]
result_bytes = base64.b64decode(result_b64)

# Save result
result_uri = store_to_gcs(
    folder="vto_results",
    file_name="tryon_result.png",
    mime_type="image/png",
    contents=result_bytes
)
```

#### Scenario B: Multiple Outfits Concurrently

```python
import concurrent.futures

def try_on_outfit(person_b64, outfit_data):
    """Try on a single outfit."""
    outfit_name, outfit_bytes = outfit_data
    outfit_b64 = base64.b64encode(outfit_bytes).decode()
    
    instances = [{
        "personImage": {"image": {"bytesBase64Encoded": person_b64}},
        "productImages": [{"image": {"bytesBase64Encoded": outfit_b64}}],
    }]
    
    response = client.predict(
        endpoint=model_endpoint,
        instances=instances,
        parameters={}
    )
    
    result_b64 = response.predictions[0]["bytesBase64Encoded"]
    result_bytes = base64.b64decode(result_b64)
    
    return outfit_name, result_bytes

# Prepare outfits
outfits = [
    ("red_dress", red_dress_bytes),
    ("blue_jeans_white_shirt", casual_bytes),
    ("formal_suit", formal_bytes),
]

person_b64 = base64.b64encode(person_image_bytes).decode()

# Process concurrently
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(
        lambda outfit: try_on_outfit(person_b64, outfit),
        outfits
    ))

# Save all results
for outfit_name, result_bytes in results:
    uri = store_to_gcs(
        folder="vto_results",
        file_name=f"tryon_{outfit_name}.png",
        mime_type="image/png",
        contents=result_bytes
    )
    print(f"Saved: {uri}")
```

---

## 7. Character Consistency

### Use Case: Maintain Character Across Scenes/Videos

Generate consistent character representations across multiple images or video frames.

#### Full Workflow

See `models/character_consistency.py` for complete implementation.

```python
from models.character_consistency import generate_character_video

# User provides reference images and scene description
reference_image_uris = [
    "gs://bucket/character/photo1.jpg",
    "gs://bucket/character/photo2.jpg",
]

scene_prompt = "walking through a futuristic city at night"

# This orchestrates the full 7-step workflow
for step_result in generate_character_video(
    user_email="user@example.com",
    reference_image_gcs_uris=reference_image_uris,
    scene_prompt=scene_prompt
):
    print(f"Step: {step_result.step_name}")
    print(f"Status: {step_result.status}")
    print(f"Message: {step_result.message}")
    
    if step_result.status == "complete":
        if step_result.step_name == "generate_video":
            video_uri = step_result.data["video_gcs_uri"]
            print(f"Final video: {video_uri}")
```

#### Abbreviated Version (Images Only)

```python
from models.gemini import (
    get_facial_composite_profile,
    get_natural_language_description,
    generate_final_scene_prompt,
)
from models.image_models import generate_images

# Step 1: Analyze reference images
profiles = [get_facial_composite_profile(img_bytes) for img_bytes in reference_images]
character_description = get_natural_language_description(profiles[0])

# Step 2: Generate scene prompt
prompts = generate_final_scene_prompt(character_description, scene_prompt)

# Step 3: Generate images
response = generate_images(
    model=cfg.MODEL_IMAGEN_3,
    prompt=prompts.prompt,
    number_of_images=4,
    aspect_ratio="16:9",
    negative_prompt=prompts.negative_prompt
)
```

---

## 8. Image Transformations

### Use Case: Creative Image Variations

Generate artistic transformations and style variations.

#### Scenario A: Style Transfer

```python
from models.gemini import generate_image_from_prompt_and_images

# Apply artistic style
original_uri = "gs://bucket/photo.jpg"

styles = [
    "in the style of watercolor painting with soft pastel colors",
    "as a vintage black and white photograph from the 1950s",
    "in anime art style with vibrant colors and dramatic lighting",
    "as an oil painting in the impressionist style",
]

for style_prompt in styles:
    gcs_uris, time = generate_image_from_prompt_and_images(
        prompt=style_prompt,
        images=[original_uri],
        gcs_folder="transformations",
        file_prefix="styled"
    )
```

#### Scenario B: AI-Suggested Transformations

```python
from models.gemini import generate_transformation_prompts

# Get AI suggestions for transformations
image_uri = "gs://bucket/landscape.jpg"
transformations = generate_transformation_prompts(image_uris=[image_uri])

# Generate each suggested transformation
for transformation in transformations:
    print(f"Trying: {transformation.title}")
    gcs_uris, time = generate_image_from_prompt_and_images(
        prompt=transformation.prompt,
        images=[image_uri],
        gcs_folder="ai_transformations",
        file_prefix=transformation.title.replace(" ", "_")
    )
```

#### Scenario C: Season/Time Transformations

```python
# Same scene, different times/seasons
base_image = "gs://bucket/park.jpg"

transformations = [
    "same scene in winter with snow covering everything",
    "same scene at sunset with warm golden lighting",
    "same scene at night with street lights and stars",
    "same scene in autumn with colorful fall foliage",
]

for transform_prompt in transformations:
    gcs_uris, time = generate_image_from_prompt_and_images(
        prompt=transform_prompt,
        images=[base_image],
        gcs_folder="seasonal",
        file_prefix="park"
    )
```

---

## 9. Batch Processing

### Use Case: Process Multiple Images Efficiently

Apply similar operations to multiple images.

#### Scenario A: Batch Background Replacement

```python
import concurrent.futures

def replace_background(image_uri, background_prompt):
    """Replace background for a single image."""
    image_bytes = download_from_gcs(image_uri)
    
    edited_uris = edit_image(
        model=cfg.MODEL_IMAGEN_EDITING,
        prompt=background_prompt,
        edit_mode="EDIT_MODE_BGSWAP",
        mask_mode="MASK_MODE_BACKGROUND",
        reference_image_bytes=image_bytes,
        number_of_images=1
    )
    
    return image_uri, edited_uris[0]

# List of product images
product_uris = [
    "gs://bucket/products/item1.jpg",
    "gs://bucket/products/item2.jpg",
    "gs://bucket/products/item3.jpg",
    # ... more products
]

background = "white seamless backdrop with professional studio lighting"

# Process in parallel
with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(
        lambda uri: replace_background(uri, background),
        product_uris
    ))

# Save results mapping
for original_uri, new_uri in results:
    print(f"Processed: {original_uri} -> {new_uri}")
```

#### Scenario B: Batch Resize and Aspect Ratio Conversion

```python
from PIL import Image
import io

def resize_to_square(image_uri, size=1024):
    """Download, resize to square, and re-upload."""
    image_bytes = download_from_gcs(image_uri)
    pil_image = Image.open(io.BytesIO(image_bytes))
    
    # Resize maintaining aspect ratio, then crop to square
    pil_image.thumbnail((size, size))
    
    # Crop to square
    width, height = pil_image.size
    if width != height:
        # Crop to center square
        left = (width - min(width, height)) // 2
        top = (height - min(width, height)) // 2
        right = left + min(width, height)
        bottom = top + min(width, height)
        pil_image = pil_image.crop((left, top, right, bottom))
    
    # Save
    byte_io = io.BytesIO()
    pil_image.save(byte_io, format="JPEG", quality=90)
    
    # Upload
    new_uri = store_to_gcs(
        folder="processed/square",
        file_name=f"square_{image_uri.split('/')[-1]}",
        mime_type="image/jpeg",
        contents=byte_io.getvalue()
    )
    
    return new_uri

# Process all images
square_uris = [resize_to_square(uri) for uri in product_uris]
```

---

## 10. Image Composition

### Use Case: Combine Multiple Images or Elements

Create composite images from multiple sources.

#### Scenario A: Create Before/After Comparison

```python
from PIL import Image
import io

def create_before_after(before_uri, after_uri, width=1200):
    """Create side-by-side comparison."""
    before_bytes = download_from_gcs(before_uri)
    after_bytes = download_from_gcs(after_uri)
    
    before_img = Image.open(io.BytesIO(before_bytes))
    after_img = Image.open(io.BytesIO(after_bytes))
    
    # Resize to same height
    target_height = 800
    before_img.thumbnail((width // 2, target_height))
    after_img.thumbnail((width // 2, target_height))
    
    # Create canvas
    canvas = Image.new("RGB", (width, target_height), color=(255, 255, 255))
    
    # Paste images side by side
    canvas.paste(before_img, (0, 0))
    canvas.paste(after_img, (width // 2, 0))
    
    # Add labels (optional, requires PIL ImageDraw)
    from PIL import ImageDraw, ImageFont
    draw = ImageDraw.Draw(canvas)
    # draw.text((10, 10), "Before", fill=(0, 0, 0))
    # draw.text((width // 2 + 10, 10), "After", fill=(0, 0, 0))
    
    # Save
    byte_io = io.BytesIO()
    canvas.save(byte_io, format="JPEG", quality=95)
    
    comparison_uri = store_to_gcs(
        folder="comparisons",
        file_name="before_after.jpg",
        mime_type="image/jpeg",
        contents=byte_io.getvalue()
    )
    
    return comparison_uri
```

#### Scenario B: Create Image Grid

```python
def create_image_grid(image_uris, cols=3, cell_size=400):
    """Create a grid of images."""
    images = [Image.open(io.BytesIO(download_from_gcs(uri))) for uri in image_uris]
    
    # Resize all images to cell_size
    for img in images:
        img.thumbnail((cell_size, cell_size))
    
    # Calculate grid dimensions
    rows = (len(images) + cols - 1) // cols
    grid_width = cols * cell_size
    grid_height = rows * cell_size
    
    # Create canvas
    grid = Image.new("RGB", (grid_width, grid_height), color=(255, 255, 255))
    
    # Paste images
    for idx, img in enumerate(images):
        row = idx // cols
        col = idx % cols
        x = col * cell_size
        y = row * cell_size
        # Center image in cell
        img_w, img_h = img.size
        offset_x = (cell_size - img_w) // 2
        offset_y = (cell_size - img_h) // 2
        grid.paste(img, (x + offset_x, y + offset_y))
    
    # Save
    byte_io = io.BytesIO()
    grid.save(byte_io, format="JPEG", quality=90)
    
    grid_uri = store_to_gcs(
        folder="grids",
        file_name="image_grid.jpg",
        mime_type="image/jpeg",
        contents=byte_io.getvalue()
    )
    
    return grid_uri

# Example usage
generated_image_uris = [
    "gs://bucket/gen1.jpg",
    "gs://bucket/gen2.jpg",
    "gs://bucket/gen3.jpg",
    "gs://bucket/gen4.jpg",
]

grid_uri = create_image_grid(generated_image_uris, cols=2)
```

#### Scenario C: Add Watermark or Logo

```python
from PIL import Image, ImageDraw

def add_watermark(image_uri, watermark_text="© 2025", position="bottom-right"):
    """Add text watermark to image."""
    image_bytes = download_from_gcs(image_uri)
    img = Image.open(io.BytesIO(image_bytes))
    
    # Create transparent overlay
    watermark = Image.new("RGBA", img.size, (255, 255, 255, 0))
    draw = ImageDraw.Draw(watermark)
    
    # Calculate position
    text_width = len(watermark_text) * 10  # Approximate
    text_height = 20
    
    if position == "bottom-right":
        x = img.width - text_width - 10
        y = img.height - text_height - 10
    elif position == "bottom-left":
        x = 10
        y = img.height - text_height - 10
    # Add more positions as needed
    
    # Draw text (semi-transparent white)
    draw.text((x, y), watermark_text, fill=(255, 255, 255, 128))
    
    # Composite
    if img.mode != "RGBA":
        img = img.convert("RGBA")
    watermarked = Image.alpha_composite(img, watermark)
    
    # Convert back to RGB for JPEG
    watermarked = watermarked.convert("RGB")
    
    # Save
    byte_io = io.BytesIO()
    watermarked.save(byte_io, format="JPEG", quality=95)
    
    result_uri = store_to_gcs(
        folder="watermarked",
        file_name=f"wm_{image_uri.split('/')[-1]}",
        mime_type="image/jpeg",
        contents=byte_io.getvalue()
    )
    
    return result_uri
```

---

## Additional Tips

### Error Handling

Always wrap API calls in try-except blocks:

```python
try:
    edited_uris = edit_image(...)
except Exception as e:
    print(f"Error editing image: {e}")
    # Fallback or retry logic
```

### Cost Optimization

1. **Use fast models for prototyping**
2. **Generate multiple variations in one call** (when supported)
3. **Cache results** in GCS for reuse
4. **Resize large images** before processing

### Quality Improvements

1. **Be specific in prompts**: Include details about lighting, style, composition
2. **Use negative prompts**: Exclude unwanted elements
3. **Try multiple samples**: Generate 2-4 options and select best
4. **Iterate**: Use generated images as references for refinement

### Performance

1. **Parallel processing**: Use ThreadPoolExecutor for multiple images
2. **Async operations**: For long-running tasks
3. **Appropriate timeouts**: Set realistic expectations for API calls
4. **Progress tracking**: Provide user feedback for long operations

---

## Related Documentation

- [Comprehensive Report](./comprehensive-image-processing-report.md)
- [Quick Reference](./quick-reference.md)
- [Main README](./README.md)

---

**Last Updated:** 2025-01-22
