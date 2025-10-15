Your task is to automatically convert a "product listicle" PDF into two CSV files for Webflow.

⚡ KEY LESSONS LEARNED:

1️⃣ URL EXTRACTION:
Image URLs and links are NOT visible as text in the PDF.
They are embedded as hyperlinks. ALWAYS use:
  strings "file.pdf" | grep "/URI"
to extract them before processing the content.

2️⃣ CRITICAL CSV FORMAT:
⚠️ COMMON ERROR: Fields shift if content has commas.
SOLUTION: ALWAYS use double quotes to enclose fields with commas.
Example: "content with, commas, and more text"
VALIDATION: Count commas in header vs data - MUST BE EQUAL.

🔧 REQUIRED TOOL: 
   - Python with csv module (import csv)
   - Use csv.writer() with quoting=csv.QUOTE_MINIMAL
   - Use csv.reader() for validation
   - DO NOT try to generate CSVs manually with string concatenation

3️⃣ PERCENT FIELDS (Products CSV):
⚠️ IMPORTANT: Each numeric rating field has its corresponding _percent field.
CALCULATION: Multiply the value by 10 (example: 9.8 → 98)
Fields with percent:
   - product_total_ranking → product_total_ranking_percent
   - product_ingredient_quality → product_ingredient_quality_percent  
   - product_flavor → product_flavor_percent
   - product_scent → product_scent_percent
   - product_value_for_money → product_value_for_money_precent (intentional typo)
⚠️ Note: "precent" in the last field maintains the typo for compatibility.

4️⃣ Learn the pattern from example files:
   - Best5s.csv (empty template)
   - Products.csv (empty template)
   - Best5sSample.csv
   - ProductsSample.csv
   - Sample.pdf

5️⃣ Once you understand the pattern, when I give you a new PDF:
   - Generate best5s_[topic].csv and products_[topic].csv following the same columns, order and format.
   - Respect literal text, don't summarize or change words.
   - ⚠️ EXCEPTION: The lede_paragraph field must have MAXIMUM 150 characters.
     If the original text is longer, summarize maintaining the essence of the message.
   - Use <p>…</p> for paragraphs in long fields (no \n).
   - Keep bullets as text with ● symbol.
   - If a field doesn't appear, leave it empty (never delete columns).
   - Save CSVs in the /csv generados/ folder.

6️⃣ URL and Image Extraction:
   
   🔍 STEP 1: Extract embedded URLs from PDF
   URLs are EMBEDDED HYPERLINKS, not visible text.
   
   Key command:
   strings "file.pdf" | grep "/URI" | grep -E "https?://"
   
   This will reveal:
   - Hero image URLs (near [source] of H1)
   - Product image URLs (each [source] per product)
   - Purchase URLs (Buy Now →)
   - Top Pick URLs (image and CTA)
   
   📋 MAPPING of found URLs:
   - Hero image → hero_image_url field (Best5s CSV)
   - Product #1 image → product_image_url field (Products CSV)
   - Product #1 purchase → product_buy_url field (Products CSV)
   - Products #2-5 (repeat the pattern)
   - Top Pick image → top_pick_image_url field (Best5s CSV)
   - Top Pick CTA → top_pick_cta_url field (Best5s CSV)
   
   ⚠️ IMPORTANT: 
   - ALWAYS search for embedded URLs first with strings
   - pdftotext does NOT show hyperlink URLs
   - The [source] markers indicate WHERE they are, but the URL is embedded
   - Webflow only accepts public URLs (https://...)
   - If there are no embedded URLs, leave fields empty

7️⃣ Before generating:
   - Count how many bullets, ingredients, checklist and pillars there really are.
   - Fill only the existing ones, leaving the others empty.

8️⃣ Text cleaning:
   - Join separated letters ("H e m p  O i l" → "Hemp Oil").
   - Keep bullets, but no HTML in them.

9️⃣ Ratings (Top 5 products):
   - Letters: F, D-, D, D+, C-, C, C+, B-, B, B+, A, A+.
   - Descending order (cannot improve going down).
   - Product #1 → 4 sub-ratings (Ingredient Quality, Flavor, Scent, Value for Money).
   - #2–#5 → 3 sub-ratings (Flavor, Scent, Value for Money).
   - Coherent and descending averages, rounded to 1 decimal.

🔟 Final validations:
   - Same number of columns as header.
   - No invented fields.
   - ⚠️ CRITICAL VALIDATION: Count commas in header vs data - MUST BE EQUAL
   - ⚠️ Double quotes in fields with commas to avoid shifting
   - ⚠️ The lede_paragraph field must have maximum 150 characters
   
   ✅ URLs that MUST be complete (if they exist in the PDF):
   Best5s CSV:
   - hero_image_url
   - top_pick_image_url
   - top_pick_cta_url
   
   Products CSV (for EACH product):
   - product_image_url
   - product_buy_url
   
   - If embedded URLs do NOT exist: leave fields empty
   - CSVs ready to import to Webflow without additional edits

1️⃣1️⃣ Final output:
   - Two correctly formatted CSVs with ALL URLs incorporated
   - Detailed report showing:
     • URLs found and mapped
     • Validations completed
     • Status: ready to import to Webflow
   - Cleanup: delete temporary files generated during the process

═══════════════════════════════════════════════════════════════

📝 COMPLETE FLOW EXAMPLE:

1. Receive PDF → "OnePet - Honest Paws Mobility Oil.pdf"

2. Extract embedded URLs:
   strings "file.pdf" | grep "/URI" | sort -u
   
   Result:
   ✓ Hero: https://cdn.prod.website-files.com/.../Group%2098.svg
   ✓ Prod #1 img: https://www.honestpaws.com/.../mobility-oil.png
   ✓ Prod #1 buy: https://www.honestpaws.com/
   ✓ (etc. for all products)

3. Extract text:
   pdftotext "file.pdf" - | grep -v "URL"
   (We already have URLs from step 2)

4. Generate CSVs with incorporated URLs:
   - best5s_honestpaws-mobility.csv ← with complete hero_image_url
   - products_honestpaws-mobility.csv ← with product_image_url and product_buy_url
   - ⚠️ USE DOUBLE QUOTES in fields with commas to avoid shifting
   - ⚠️ VALIDATE: commas in header = commas in data
   
   🔧 CORRECT TECHNICAL METHOD (use Python with csv module):
   
   ```python
   import csv
   
   # Header as string (WITHOUT quotes)
   header_line = "Name,Slug,Collection ID,Locale ID,..."
   
   # Data as list of values
   row = ["Honest Paws", "slug", "", "", "text with, commas", ...]
   
   # Write CSV correctly
   with open(output_file, 'w', encoding='utf-8', newline='') as f:
       # 1. Header without quotes
       f.write(header_line + '\n')
       
       # 2. Data with csv.writer and QUOTE_MINIMAL
       #    (adds quotes ONLY where necessary)
       writer = csv.writer(f, quoting=csv.QUOTE_MINIMAL)
       writer.writerow(row)
   ```
   
   ⚠️ DO NOT use manual string concatenation methods
   ⚠️ DO NOT use echo/cat to write CSVs
   ⚠️ The csv module automatically handles quotes and escapes

5. Validate:
   
   🔧 CORRECT VALIDATION (use csv.reader):
   
   ```python
   import csv
   
   with open(csv_file, 'r', encoding='utf-8') as f:
       reader = csv.reader(f)
       lines = list(reader)
       
       header = lines[0]
       data = lines[1]
       
       print(f"Header columns: {len(header)}")
       print(f"Data columns: {len(data)}")
       
       if len(header) == len(data):
           print("✅ VALIDATION SUCCESSFUL")
       else:
           print("❌ ERROR in number of columns")
   ```
   
   ⚠️ DO NOT use 'tr -cd "," | wc -c' (counts commas inside quotes)
   ⚠️ csv.reader understands CSV format correctly
   ✓ 64 columns in Best5s
   ✓ 30 columns in Products (includes 5 _percent fields)
   ✓ All URLs present
   ✓ 5 products with descending ratings

6. Result: Files ready to import to Webflow 🚀
