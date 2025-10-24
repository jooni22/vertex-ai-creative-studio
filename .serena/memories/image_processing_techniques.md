# Image Processing Techniques in Repository

## PIL/Pillow Operations (from character_consistency.py)
1. **Image Loading**: `PIL_Image.open(io.BytesIO(image_bytes))`
2. **Size Analysis**: `width, height = pil_image.size`
3. **Resizing**: `pil_image.thumbnail(target_size)`
4. **Padding**: Custom `_pad_to_target_size()` function
5. **Mask Creation**: `PIL_Image.new("L", size, color)`
6. **Compositing**: `image.paste(source, (x, y))`
7. **Format Conversion**: `.convert('RGB')`, `.convert('L')`
8. **Bytes Handling**: `io.BytesIO()` for in-memory operations

## Mask Modes Available
- `MASK_MODE_FOREGROUND` - Auto-detect foreground objects
- `MASK_MODE_BACKGROUND` - Auto-detect background
- `MASK_MODE_SEMANTIC` - Use 194 semantic classes
- `MASK_MODE_PROMPT` - Text description for mask
- `MASK_MODE_USER_PROVIDED` - Custom mask upload

## Edit Modes (Imagen)
- `EDIT_MODE_INPAINT_INSERTION` - Add objects
- `EDIT_MODE_INPAINT_REMOVAL` - Remove objects
- `EDIT_MODE_BGSWAP` - Replace background
- `EDIT_MODE_OUTPAINT` - Extend image boundaries
- `EDIT_MODE_DEFAULT` - Mask-free editing

## Semantic Classes (194 total)
Key classes for fashion: person (125), shirt, jacket, dress, pants, etc.
Full list in `/components/constants.py` lines 147-288

## Image Processing Pipeline Examples
1. **Character Consistency** (7 steps):
   - Face analysis → Description → Scene prompt → Image gen → Selection → Outpaint → Video
   
2. **Product Recontextualization**:
   - Product extraction → Scene generation → Quality evaluation
   
3. **Virtual Try-On at Scale**:
   - Concurrent processing → Side-by-side comparison → Batch generation
