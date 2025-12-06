#!/usr/bin/env python3
"""
Genesis Channel Badge Generator
Generates 333 numbered Platinum tier badges and Crossmint CSV file
"""

from PIL import Image, ImageDraw, ImageFont
import csv
import os

# Configuration
TEMPLATE_PATH = "Genesis_Channel_Platinum_2000_blank.png"
OUTPUT_DIR = "genesis_badges_output"
CSV_FILENAME = "genesis_channels_platinum.csv"

# Text styling (adjust these to match your original design)
TEXT_POSITION = (280, 580)  # X, Y coordinates for "NODE" text
NUMBER_POSITION = (280, 630)  # X, Y coordinates for "#001" text
TEXT_COLOR = (180, 180, 180)  # Light gray color (RGB)
FONT_SIZE_NODE = 48  # Size for "NODE" text
FONT_SIZE_NUMBER = 72  # Size for "#001" text

# Badge metadata
TIER = "Platinum"
TOTAL_BADGES = 333
PRICE = 500
CAPACITY = "100/hour"
ROUTING_WEIGHT = "1x"
BASE_URL = "https://0l1labs.com"
IMAGE_BASE_URL = "ipfs://YOUR_IPFS_CID_HERE"  # Update after IPFS upload

def setup_output_directory():
    """Create output directory if it doesn't exist"""
    if not os.path.exists(OUTPUT_DIR):
        os.makedirs(OUTPUT_DIR)
        print(f"✓ Created output directory: {OUTPUT_DIR}")

def generate_badge(template_path, node_number, output_path):
    """Generate a single numbered badge"""
    # Load template
    img = Image.open(template_path)
    draw = ImageDraw.Draw(img)
    
    # Try to use a nice font, fallback to default if not available
    try:
        font_node = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf", FONT_SIZE_NODE)
        font_number = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf", FONT_SIZE_NUMBER)
    except:
        print("⚠ Using default font (install custom fonts for better results)")
        font_node = ImageFont.load_default()
        font_number = ImageFont.load_default()
    
    # Add "NODE" text
    draw.text(TEXT_POSITION, "NODE", fill=TEXT_COLOR, font=font_node)
    
    # Add number (formatted as #001, #002, etc.)
    number_text = f"#{node_number:03d}"
    draw.text(NUMBER_POSITION, number_text, fill=TEXT_COLOR, font=font_number)
    
    # Save the image
    img.save(output_path, "PNG", optimize=True)
    
def generate_all_badges():
    """Generate all 333 badges"""
    print(f"\n🎨 Generating {TOTAL_BADGES} Genesis Channel badges...\n")
    
    for i in range(1, TOTAL_BADGES + 1):
        output_filename = f"platinum-{i:03d}.png"
        output_path = os.path.join(OUTPUT_DIR, output_filename)
        
        generate_badge(TEMPLATE_PATH, i, output_path)
        
        # Progress indicator
        if i % 50 == 0:
            print(f"✓ Generated {i}/{TOTAL_BADGES} badges...")
    
    print(f"\n✓ All {TOTAL_BADGES} badges generated successfully!")

def generate_csv():
    """Generate CSV file for Crossmint batch upload"""
    csv_path = os.path.join(OUTPUT_DIR, CSV_FILENAME)
    
    print(f"\n📄 Generating CSV file: {CSV_FILENAME}...\n")
    
    with open(csv_path, 'w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['name', 'description', 'image', 'external_url', 'attributes']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        
        writer.writeheader()
        
        for i in range(1, TOTAL_BADGES + 1):
            # Format number with leading zeros
            num = f"{i:03d}"
            
            # Create attributes JSON
            attributes = [
                {"trait_type": "Tier", "value": TIER},
                {"trait_type": "Channel Number", "value": num},
                {"trait_type": "Capacity", "value": CAPACITY},
                {"trait_type": "Routing Weight", "value": ROUTING_WEIGHT},
                {"trait_type": "Series", "value": "Genesis"}
            ]
            
            # Convert attributes to JSON string format
            attributes_str = str(attributes).replace("'", '"')
            
            row = {
                'name': f'0L1 Labs Genesis Channel #{num}',
                'description': f'Genesis Channel #{num} - Platinum Tier. One of 333 infrastructure nodes earning 70% of verification fees. {CAPACITY} verification capacity. Includes channel API access and transferable revenue rights.',
                'image': f'{IMAGE_BASE_URL}/platinum-{num}.png',
                'external_url': BASE_URL,
                'attributes': attributes_str
            }
            
            writer.writerow(row)
    
    print(f"✓ CSV file generated: {csv_path}")
    print(f"✓ Contains {TOTAL_BADGES} badge entries")

def main():
    """Main execution"""
    print("=" * 60)
    print("GENESIS CHANNEL BADGE GENERATOR")
    print("0L1 Labs - Platinum Tier")
    print("=" * 60)
    
    # Check if template exists
    if not os.path.exists(TEMPLATE_PATH):
        print(f"\n❌ ERROR: Template file not found: {TEMPLATE_PATH}")
        print("Please ensure the blank template is in the same directory.")
        return
    
    # Setup
    setup_output_directory()
    
    # Generate badges
    generate_all_badges()
    
    # Generate CSV
    generate_csv()
    
    print("\n" + "=" * 60)
    print("✓ GENERATION COMPLETE!")
    print("=" * 60)
    print(f"\nOutput location: ./{OUTPUT_DIR}/")
    print(f"  • {TOTAL_BADGES} badge images (platinum-001.png to platinum-{TOTAL_BADGES:03d}.png)")
    print(f"  • 1 CSV file ({CSV_FILENAME})")
    print("\nNext steps:")
    print("1. Upload all images to IPFS (use Pinata or NFT.Storage)")
    print("2. Update IMAGE_BASE_URL in this script with your IPFS CID")
    print("3. Re-run to regenerate CSV with correct image URLs")
    print("4. Upload CSV to Crossmint for batch minting")
    print("\n")

if __name__ == "__main__":
    main()
