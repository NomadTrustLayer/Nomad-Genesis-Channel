# Genesis Channel Badge Generator - Instructions

## Quick Start

1. **Install Python & Dependencies**
   ```bash
   pip install Pillow
   ```

2. **Place Files in Same Directory**
   - `generate_genesis_badges.py` (the script)
   - `Genesis_Channel_Platinum_2000_blank.png` (your template)

3. **Run the Script**
   ```bash
   python3 generate_genesis_badges.py
   ```

4. **Output**
   - Creates `genesis_badges_output/` folder
   - Generates 333 badge images: `platinum-001.png` to `platinum-333.png`
   - Generates CSV file: `genesis_channels_platinum.csv`

---

## Important: Adjust Text Position

The script uses default positions for the "NODE" and "#001" text.

**You'll need to adjust these coordinates to match your original design:**

Open `generate_genesis_badges.py` and modify:
```python
TEXT_POSITION = (280, 580)      # X, Y for "NODE" 
NUMBER_POSITION = (280, 630)    # X, Y for "#001"
```

**How to find the right coordinates:**
1. Open your original `Genesis_Channel_Platinum_2000.png` in Photoshop
2. Select the text tool and hover over where "NODE" starts
3. Note the X, Y coordinates (shown in Photoshop's info panel)
4. Update the script with those coordinates
5. Run again to regenerate

---

## Text Styling Adjustments

If the text doesn't match your original style, adjust:

```python
TEXT_COLOR = (180, 180, 180)    # RGB color (currently light gray)
FONT_SIZE_NODE = 48             # "NODE" text size
FONT_SIZE_NUMBER = 72           # "#001" number size
```

**For better font matching:**
- The script uses DejaVu Sans Bold by default
- To use a specific font, download it and update the font path:
  ```python
  font_node = ImageFont.truetype("/path/to/your/font.ttf", FONT_SIZE_NODE)
  ```

---

## After Generating Badges

### Step 1: Upload to IPFS

**Option A - Pinata (Recommended)**
1. Go to https://pinata.cloud
2. Create free account
3. Upload entire `genesis_badges_output` folder
4. Get IPFS CID (e.g., `QmXxx...`)

**Option B - NFT.Storage**
1. Go to https://nft.storage
2. Upload folder
3. Get IPFS CID

### Step 2: Update CSV with IPFS URLs

1. Open `generate_genesis_badges.py`
2. Find this line:
   ```python
   IMAGE_BASE_URL = "ipfs://YOUR_IPFS_CID_HERE"
   ```
3. Replace with your actual CID:
   ```python
   IMAGE_BASE_URL = "ipfs://QmXxx..."  # Your IPFS CID
   ```
4. Run script again (only regenerates CSV, doesn't remake images)

### Step 3: Upload to Crossmint

1. Log into Crossmint dashboard
2. Navigate to your Genesis Channels collection
3. Upload the CSV file
4. Review and confirm
5. Set pricing ($500)
6. Enable minting

---

## Troubleshooting

**Problem: Text is in wrong position**
- Solution: Adjust `TEXT_POSITION` and `NUMBER_POSITION` coordinates

**Problem: Text color doesn't match**
- Solution: Adjust `TEXT_COLOR` RGB values

**Problem: Font looks wrong**
- Solution: Install custom font and update font path

**Problem: Script fails with "Template not found"**
- Solution: Ensure blank template is in same directory and named correctly

**Problem: Generated images too large**
- Solution: Images are optimized PNG, should be 1-3MB each (acceptable for NFTs)

---

## CSV Format

The generated CSV includes:
- **name**: "0L1 Labs Genesis Channel #001"
- **description**: Full badge description with tier info
- **image**: IPFS URL to badge image
- **external_url**: Link to 0l1labs.com
- **attributes**: JSON with traits (Tier, Channel Number, Capacity, etc.)

---

## Need Help?

If something doesn't work:
1. Check Python version (needs 3.6+)
2. Verify Pillow is installed: `pip list | grep Pillow`
3. Make sure template file is in correct location
4. Check console output for error messages

---

## Total Generation Time

- **Badge generation**: ~2-5 minutes (333 images)
- **CSV generation**: ~1 second
- **IPFS upload**: ~5-10 minutes (depending on connection)

**Total**: ~10-15 minutes from start to Crossmint-ready

---

Good luck with the launch! 🚀
