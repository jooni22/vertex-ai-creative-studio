# Ghost Mannequin Pipeline - Project Summary

## Goal
Create ghost mannequin effect images from 2-3 garment photos using ONLY open-source models.

## Key Requirements
- ❌ NO Google Cloud APIs (Imagen, Gemini, VTO)
- ✅ Only open-source models
- Input: 2-3 photos (front, back, optionally inside)
- Output: Professional ghost mannequin composite

## 5-Stage Pipeline

### Stage 1: Segmentation
- **Model**: U2NET (already in `docs/cloth_segment/`)
- **Additional**: Rembg for background removal
- **Output**: Clean garment with alpha masks

### Stage 2: Alignment
- **Tools**: OpenCV SIFT/ORB feature matching
- **Process**: Homography transformation to align front/back
- **Output**: Aligned and scaled images

### Stage 3: Inpainting
- **Model**: LaMa (Resolution-robust Large Mask Inpainting)
- **Alternative**: Stable Diffusion Inpainting
- **Output**: Filled gaps where mannequin was removed

### Stage 4: Compositing
- **Tools**: PIL/Pillow + OpenCV
- **Process**: Alpha blending, hollow center creation
- **Output**: Combined front + back with ghost effect

### Stage 5: Post-Processing
- **Tools**: OpenCV, scikit-image, Real-ESRGAN (optional)
- **Process**: Color matching, seam blending, enhancement
- **Output**: Professional quality final image

## Technology Stack
1. U2NET - cloth segmentation (✅ ready)
2. Rembg - background removal
3. LaMa - inpainting
4. OpenCV - alignment, blending
5. PIL/Pillow - compositing
6. Real-ESRGAN - upscaling (optional)

## Implementation Phases
- Phase 1: MVP CLI tool (2-3 weeks)
- Phase 2: Quality improvements (2-3 weeks)
- Phase 3: Web UI (2-3 weeks)
- Phase 4: Advanced features (3-4 weeks)

## Key Files Created
- `docs/ghost-mannequin-pipeline-plan.md` - Complete implementation plan
