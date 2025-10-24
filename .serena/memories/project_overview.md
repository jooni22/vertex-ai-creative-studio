# Vertex AI Creative Studio - Project Overview

## Purpose
Web application showcasing Google Cloud's generative media capabilities including:
- Image: Imagen 3/4, Virtual Try-On, Gemini 2.5 Flash Image Generation
- Video: Veo 2, Veo 3
- Music: Lyria
- Speech: Chirp 3 HD, Gemini TTS
- Workflows: Character Consistency, Shop the Look, Product Recontextualization

## Tech Stack
- **Framework**: Mesop (Google's Python framework for rapid AI app development)
- **Cloud**: Google Cloud Platform, Vertex AI
- **Image Processing**: PIL/Pillow, OpenCV, NumPy
- **AI Models**: Imagen, Gemini, Veo, Lyria, Chirp
- **Deployment**: Cloud Run, Terraform, Cloud Build

## Project Structure
- `/models/` - Core AI model integrations (image_models.py, gemini.py, vto.py)
- `/pages/` - UI pages for different features
- `/experiments/` - Standalone applications and new features
- `/components/` - Reusable UI components and constants
- `/config/` - Configuration files (default.py)
- `/docs/` - Documentation and analysis
  - `/docs/cloth_segment/` - U2NET cloth segmentation (ready to use)
  - `/docs/image-processing-capabilities/` - Comprehensive image processing report

## Key Capabilities
1. **Image Generation**: Text-to-image with Imagen 3/4
2. **Image Editing**: Inpainting, outpainting, background swap, mask-based editing
3. **Virtual Try-On**: Fashion/apparel placement on person images
4. **Product Recontextualization**: Place products in new scenes
5. **Character Consistency**: Multi-step workflow with PIL operations
6. **Cloth Segmentation**: U2NET model for garment parsing (3 classes)

## Development Commands
- Run dev server: `python main.py` or use PM2 for background
- Install deps: `pip install -r requirements.txt`
- Format: Uses pyproject.toml configuration
