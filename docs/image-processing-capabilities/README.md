# Image Processing Capabilities Documentation

This folder contains comprehensive documentation of image processing, editing, and generation capabilities demonstrated in the vertex-ai-creative-studio repository.

## Documents

### 📄 [Comprehensive Image Processing Report](./comprehensive-image-processing-report.md)

A detailed analysis covering:

- **Google Cloud Image Models**: Imagen 3/4, Gemini 2.5 Flash Image, Virtual Try-On
- **Image Processing Libraries**: PIL/Pillow, OpenCV, NumPy, scikit-image
- **Mask Generation & Segmentation**: Automatic mask modes, semantic segmentation (194 classes)
- **LLM-Based Capabilities**: Gemini-powered workflows and transformations
- **Code Examples**: Production-ready implementations from the repository
- **Open-Source Alternatives**: Comparison with community models

### 📋 [Quick Reference Guide](./quick-reference.md)

Essential information at a glance:
- Model IDs and capabilities
- Edit modes and mask modes
- Semantic segmentation classes
- Common code patterns

### 🔍 [Use Case Examples](./use-case-examples.md)

Practical examples for common scenarios:
- Background removal and replacement
- Object insertion and removal
- Image expansion (outpainting)
- Product photography enhancement
- Fashion try-on
- Character consistency in video

## Key Findings

### Google Models Available

| Model Family | Primary Use | Key Features |
|-------------|-------------|--------------|
| **Imagen 3/4** | Text-to-image generation | Multiple variants (standard, fast, ultra), aspect ratios |
| **Imagen Editing** | Image manipulation | Inpainting, outpainting, background swap, mask-free editing |
| **Gemini 2.5 Flash Image** | Multimodal generation | Reference-guided, conversational, transformation suggestions |
| **Virtual Try-On** | Fashion/apparel | Realistic clothing placement on person images |
| **Imagen Product Recontext** | Product placement | Place products in new scenes/contexts |

### Image Processing Capabilities

#### With AI Models:
- ✅ Text-to-image generation
- ✅ Image editing with masks (foreground, background, semantic)
- ✅ Automatic object segmentation (194 classes)
- ✅ Image outpainting and inpainting
- ✅ Background removal/replacement
- ✅ Object insertion and removal
- ✅ Virtual try-on for clothing
- ✅ Product recontextualization
- ✅ Style transfer and transformations

#### With Python Libraries (PIL, OpenCV):
- ✅ Image loading, saving, format conversion
- ✅ Resizing, cropping, padding
- ✅ Mask creation and manipulation
- ✅ Aspect ratio calculations
- ✅ Image composition and blending
- ✅ Video frame extraction
- ✅ Color space conversions
- ✅ Drawing and text overlay

### Complex Workflows

The repository demonstrates sophisticated multi-step workflows:

1. **Character Consistency** (7 steps)
   - Facial analysis → Description generation → Scene prompting → Image generation → Selection → Outpainting → Video creation

2. **Product Recontextualization**
   - Product extraction → Scene generation → Quality evaluation

3. **Virtual Try-On at Scale**
   - Concurrent processing → Side-by-side comparison → Batch generation

## Quick Start

### Generate an Image with Imagen

```python
from models.image_models import generate_images

response = generate_images(
    model="imagen-3.0-fast-generate-001",
    prompt="a serene mountain landscape at sunset",
    number_of_images=1,
    aspect_ratio="16:9",
    negative_prompt="people, buildings, cars"
)
```

### Edit an Image

```python
from models.image_models import edit_image

edited_uris = edit_image(
    model="imagen-3.0-capability-001",
    prompt="add a rainbow in the sky",
    edit_mode="EDIT_MODE_INPAINT_INSERTION",
    mask_mode="MASK_MODE_BACKGROUND",
    reference_image_bytes=image_bytes,
    number_of_images=1
)
```

### Process an Image with PIL

```python
from PIL import Image
import io

# Load image
pil_image = Image.open(io.BytesIO(image_bytes))

# Get dimensions
width, height = pil_image.size

# Resize while maintaining aspect ratio
pil_image.thumbnail((1024, 1024))

# Create a mask
mask = Image.new("L", pil_image.size, 0)

# Save to bytes
byte_io = io.BytesIO()
pil_image.save(byte_io, "PNG")
image_bytes = byte_io.getvalue()
```

## File References

Key files in the repository for image processing:

### Core Implementation
- `models/image_models.py` - Imagen generation and editing
- `models/gemini.py` - Gemini image capabilities
- `models/vto.py` - Virtual Try-On
- `models/character_consistency.py` - Complex workflow with PIL

### Configuration
- `config/default.py` - Model IDs and settings
- `components/constants.py` - Edit/mask modes, semantic classes

### UI Pages
- `pages/imagen.py` - Imagen generation interface
- `pages/edit_images.py` - Image editing interface
- `pages/gemini_image_generation.py` - Gemini image interface

### Experiments
- `experiments/VTO/VTOatScale.ipynb` - Virtual Try-On at scale
- `experiments/Imagen_Product_Recontext/` - Product recontextualization
- `experiments/veo3-character-consistency/workflow.ipynb` - Character workflow

## Semantic Segmentation Classes

The Imagen editing API supports automatic segmentation for 194 object types, including:

- **People & Body Parts**: person (125), rider (126-128)
- **Animals**: dog (8), cat (7), horse (9), bird (6), elephant (12), etc.
- **Vehicles**: car (176), bicycle (175), motorcycle (178), airplane (179), etc.
- **Furniture**: chair (57-59), table (67), bed (66), couch (62), etc.
- **Electronics**: laptop (37), television (42), cell phone (41), etc.
- **Food**: pizza (52), cake (54), apple (46), banana (45), etc.
- **Nature**: vegetation (174), mountain/hill (147), water (186-188), etc.

See [comprehensive report](./comprehensive-image-processing-report.md#appendix-a-semantic-segmentation-class-ids) for the complete list.

## Edit Modes

| Mode | Value | Purpose |
|------|-------|---------|
| **Insert** | `EDIT_MODE_INPAINT_INSERTION` | Add new objects/content |
| **Remove** | `EDIT_MODE_INPAINT_REMOVAL` | Erase objects |
| **Background Swap** | `EDIT_MODE_BGSWAP` | Replace background |
| **Outpaint** | `EDIT_MODE_OUTPAINT` | Extend image boundaries |
| **Default** | `EDIT_MODE_DEFAULT` | General mask-free edits |

## Mask Modes

| Mode | Value | Purpose |
|------|-------|---------|
| **Foreground** | `MASK_MODE_FOREGROUND` | Auto-detect main subjects |
| **Background** | `MASK_MODE_BACKGROUND` | Auto-detect background |
| **Semantic** | `MASK_MODE_SEMANTIC` | Use object class IDs |
| **Prompt** | `MASK_MODE_PROMPT` | Describe mask area |
| **User Provided** | `MASK_MODE_USER_PROVIDED` | Supply custom mask |

## Resources

### Google Cloud Documentation
- [Vertex AI Imagen Overview](https://cloud.google.com/vertex-ai/generative-ai/docs/image/overview)
- [Imagen Image Generation](https://cloud.google.com/vertex-ai/generative-ai/docs/image/generate-images)
- [Imagen Image Editing](https://cloud.google.com/vertex-ai/generative-ai/docs/image/edit-images)
- [Virtual Try-On](https://cloud.google.com/vertex-ai/generative-ai/docs/image/virtual-try-on)

### Python Libraries
- [Pillow Documentation](https://pillow.readthedocs.io/)
- [OpenCV Documentation](https://docs.opencv.org/)
- [scikit-image](https://scikit-image.org/)
- [NumPy](https://numpy.org/doc/)

### Open-Source Models
- [Hugging Face Models](https://huggingface.co/models)
- [Segment Anything (SAM)](https://github.com/facebookresearch/segment-anything)
- [Stable Diffusion](https://github.com/Stability-AI/stablediffusion)
- [ControlNet](https://github.com/lllyasviel/ControlNet)

## Contributing

Found something missing or want to add more examples? Please contribute!

1. Review the comprehensive report
2. Check existing examples
3. Add new use cases or clarifications
4. Submit improvements

## License

This documentation is part of the vertex-ai-creative-studio repository and follows the same Apache 2.0 license.

---

**Last Updated:** 2025-01-22  
**Maintained by:** Repository Contributors
