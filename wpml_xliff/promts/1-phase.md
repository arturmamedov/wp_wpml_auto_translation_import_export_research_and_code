# XLIFF Translation Automation - Phase 1: XLIFF Parser Foundation

## **Project Overview**
I'm building a comprehensive XLIFF translation system for Nests Hostels that will:
- Process WPML-exported XLIFF files 
- Translate content via OpenAI API while preserving Gen-Z/Millennial brand voice
- Maintain all WordPress formatting (shortcodes, HTML, SEO metadata)
- Output importable XLIFF files for seamless WPML workflow
- Handle 60 posts → 300 translations (Spanish → English, German, French, Italian)

## **Current Phase Goal**
Build a robust **XLIFFParser class** that can reliably read, analyze, and reconstruct XLIFF files with perfect structure preservation. This is the foundation for the entire translation workflow.

## **Technical Approach Decision**
**Using OOP architecture** - this project has multiple interconnected components, needs state management, and will become a WordPress plugin. OOP provides better organization and maintainability.

---

## **PHASE 1 REQUIREMENTS: XLIFFParser Class**

### **Core Functionality Needed:**
1. **Parse XLIFF files** (XLIFF 1.2/2.1 compatibility)
2. **Extract translation units** with complete metadata preservation
3. **Analyze content types** (social media, website copy, SEO meta, etc.)
4. **Preserve WordPress elements** (shortcodes, HTML tags, CDATA sections, new lines characters)
5. **Reconstruct XLIFF files** with translations maintaining exact structure

### **Project Structure Setup:**
```
XLIFFTranslationWorkflow/
├── src/
│   ├── Core/
│   │   ├── XLIFFParser.php          # ← THIS IS WHAT WE'RE BUILDING
│   │   ├── BrandVoiceTranslator.php # (Phase 2)
│   │   ├── TranslationMemoryManager.php # (Phase 3)
│   │   └── XLIFFRebuilder.php       # (Phase 4)
│   └── Utils/
│       ├── Logger.php
│       └── ContentTypeDetector.php
├── config/
│   └── brand-voice-config.php
├── tests/
│   └── XLIFFParserTest.php
└── composer.json
```

---

## **XLIFFParser Class Implementation Requirements:**

### **Class Structure:**
```php
class XLIFFParser {
    private $dom;
    private $xpath;
    private $translationUnits = [];
    private $metadata = [];
    private $originalStructure = [];
    
    // Main methods to implement:
    public function parseXLIFF($xliffContent)
    public function extractTranslationUnits()
    public function analyzeContentTypes()
    public function preserveStructure()
    public function validateXLIFFFormat()
}
```

### **Key Parsing Requirements:**

**1. Handle XLIFF Structure Variations:**
- Support XLIFF 1.2 
- Parse `<trans-unit>` (1.2) and other tags
- Preserve all XML namespaces and attributes
- Handle nested HTML and CDATA sections

**2. WordPress-Specific Elements:**
- Extract post metadata (post_id, post_type, meta_key)
- Preserve shortcodes: `[gallery id="123"]`, `[contact-form-7]`, etc.
- Maintain HTML structure with attributes
- Handle special characters and encoding
- Preserve new lines characters

**3. Content Type Detection:**
- Identify content types from XLIFF metadata
- Classify as: social_media, website_copy, seo_meta, post_title, etc.
- Extract context for brand voice application

**4. Translation Unit Structure:**
Each translation unit should contain:
```php
[
    'id' => 'unique_identifier',
    'source' => 'source_text',
    'source_language' => 'es',
    'content_type' => 'website_copy', // detected type
    'post_id' => 123,
    'post_type' => 'post',
    'meta_key' => 'post_content', // or 'post_title', 'meta_description', etc.
    'html_tags' => [], // extracted HTML for preservation
    'shortcodes' => [], // extracted shortcodes
    'metadata' => [], // additional XLIFF metadata
    'context' => [] // surrounding content for translation context
]
```

---

## **Development Steps for This Phase:**

### **Step 1: Basic XLIFF Loading and Validation**
- Create XLIFFParser class with DOMDocument
- Load XLIFF file with proper error handling  
- Validate XML structure and detect XLIFF version
- Set up XPath for element navigation

### **Step 2: Translation Unit Extraction**
- Extract all `<trans-unit>` or `<unit>` elements and other tags
- Preserve all attributes and metadata
- Handle source text with HTML/shortcodes
- Build translation unit array structure

### **Step 3: Content Type Analysis**
- Analyze XLIFF metadata to detect content types
- Create ContentTypeDetector utility class
- Classify each unit for appropriate brand voice prompts
- Extract context from surrounding elements

### **Step 4: Structure Preservation System**
- Store original XML structure for reconstruction
- Create mapping system for element positions
- Preserve namespaces, attributes, and formatting
- Prepare for clean insertion of translations

### **Step 5: Testing and Validation**
- Test with real WPML XLIFF exports
- Validate structure preservation
- Test WordPress element handling
- Verify content type detection accuracy

---

## **Sample Files I Have Available:**
- Real XLIFF files from WPML exports
- Brand voice guidelines for Nests Hostels  
- Translation examples with before/after
- WordPress content with various shortcodes and HTML

---

## **Success Criteria for Phase 1:**
- ✅ Parse my sample XLIFF files without errors
- ✅ Extract all translation units with complete metadata
- ✅ Correctly detect content types (social_media, website_copy, etc.)
- ✅ Preserve all WordPress shortcodes and HTML structure  
- ✅ Maintain exact XLIFF structure for reconstruction
- ✅ Handle XLIFF 1.2
- ✅ Process files with 50+ translation units efficiently

---

## **What I Need Help With:**

1. **Analyze my XLIFF file structure** - Let's examine a real WPML export together
2. **Design the optimal parsing strategy** - Best approach for preserving everything
3. **Implement the XLIFFParser class** - Clean, efficient OOP implementation
4. **Create content type detection logic** - Classify content for brand voice prompts
5. **Build structure preservation system** - Ensure perfect reconstruction capability
6. **Set up testing framework** - Validate against real XLIFF files

---

## **Key Technical Decisions Needed:**

**XML Parsing Approach:**
- DOMDocument vs SimpleXML vs XMLReader?
- XPath strategy for complex element selection?
- Memory efficiency for large files?

**Content Type Detection:**
- Metadata-based classification vs content analysis?
- How to handle mixed content types?
- Context extraction for translation quality?

**Structure Preservation:**
- Element mapping strategy?
- Namespace handling approach?  
- Attribute preservation method?

---

**Ready to start with analyzing my sample XLIFF files and building the foundation XLIFFParser class. Let's make this bulletproof from the beginning!**

## **My Sample XLIFF File:**
I'll upload my real WPML XLIFF files of different post types (wordpress gutenberg editor page and blog post, Hostel of custom type with a lot of ACF custom fields) export so we can analyze the exact structure we're working with. Analyze all different style of content find the differences for create all necessary cases of content and it's different contents. And look at the fact that there are duplications, we will need to manage them also (it's part of this challange)....
