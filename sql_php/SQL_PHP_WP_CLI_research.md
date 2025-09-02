L'approccio SQL diretto è tecnicamente possibile e potrebbe essere la soluzione più economica, ma presenta sfide significative legate alla struttura complessa del database WPML. Ti faccio una ricerca approfondita per trovare metodi pratici e sicuri.Eccellente! Hai identificato la soluzione migliore. Un approccio ibrido **SQL + PHP + WP-CLI** è la strada più efficace e completamente gratuita. Dalla ricerca ho trovato il workflow perfetto che combina tutti i metodi:

## Approccio raccomandato: SQL Export + PHP Scripts + WPML API

**Questo metodo è completamente gratuito e automatizzabile:**

### 1. **Estrazione SQL dal sito originale**
```sql
-- Estrai post spagnoli con tutti i metadati
SELECT 
    p.ID, p.post_title, p.post_content, p.post_excerpt, p.post_name,
    pm1.meta_value as yoast_title,
    pm2.meta_value as yoast_desc,
    pm3.meta_value as yoast_focuskw
FROM wp_posts p
LEFT JOIN wp_postmeta pm1 ON p.ID = pm1.post_id AND pm1.meta_key = '_yoast_wpseo_title'
LEFT JOIN wp_postmeta pm2 ON p.ID = pm2.post_id AND pm2.meta_key = '_yoast_wpseo_metadesc'  
LEFT JOIN wp_postmeta pm3 ON p.ID = pm3.post_id AND pm3.meta_key = '_yoast_wpseo_focuskw'
WHERE p.post_type = 'post' AND p.post_status = 'publish';
```

### 2. **Script PHP per creare traduzioni programmaticamente**
```php
<?php
// Script per creare traduzioni automaticamente
class WPML_Translation_Creator {
    private $target_languages = ['en', 'de', 'it', 'fr'];
    
    public function create_translations($spanish_post_data) {
        global $sitepress;
        
        foreach ($this->target_languages as $lang) {
            // 1. Traduci il contenuto con AI API
            $translated_content = $this->translate_content($spanish_post_data, $lang);
            
            // 2. Crea il post tradotto
            $translated_post_id = wp_insert_post([
                'post_title'   => $translated_content['title'],
                'post_content' => $translated_content['content'], 
                'post_excerpt' => $translated_content['excerpt'],
                'post_status'  => 'publish',
                'post_type'    => 'post',
                'post_name'    => sanitize_title($translated_content['title'])
            ]);
            
            // 3. Aggiungi metadati Yoast SEO tradotti
            update_post_meta($translated_post_id, '_yoast_wpseo_title', $translated_content['seo_title']);
            update_post_meta($translated_post_id, '_yoast_wpseo_metadesc', $translated_content['seo_desc']);
            update_post_meta($translated_post_id, '_yoast_wpseo_focuskw', $translated_content['focus_kw']);
            
            // 4. Collega alla traduzione WPML
            $this->connect_translation($spanish_post_data['ID'], $translated_post_id, $lang);
        }
    }
    
    private function connect_translation($original_id, $translated_id, $language) {
        global $sitepress;
        
        // Ottieni trid del post originale
        $trid = apply_filters('wpml_element_trid', null, $original_id, 'post_post');
        
        if ($trid) {
            // Collega la traduzione
            do_action('wpml_set_element_language_details', [
                'element_id'    => $translated_id,
                'element_type'  => 'post_post', 
                'trid'          => $trid,
                'language_code' => $language,
                'source_language_code' => 'es'
            ]);
        }
    }
    
    private function translate_content($content, $target_lang) {
        // Integrazione con API di traduzione (DeepL, OpenAI, ecc.)
        return [
            'title' => $this->call_translation_api($content['post_title'], $target_lang),
            'content' => $this->call_translation_api($content['post_content'], $target_lang),
            'excerpt' => $this->call_translation_api($content['post_excerpt'], $target_lang),
            'seo_title' => $this->call_translation_api($content['yoast_title'], $target_lang),
            'seo_desc' => $this->call_translation_api($content['yoast_desc'], $target_lang),
            'focus_kw' => $this->call_translation_api($content['yoast_focuskw'], $target_lang)
        ];
    }
}
```

### 3. **Comando WP-CLI per finalizzare**
```bash
# Dopo aver eseguito lo script PHP, usa WP-CLI per collegare automaticamente
wp wpml import process
```

## Vantaggi di questo approccio

**✅ Completamente gratuito** - nessun plugin a pagamento
**✅ Automatizzabile** - script PHP per batch processing  
**✅ Sicuro** - usa API WordPress native invece di manipolare direttamente il database
**✅ Preserva SEO** - gestisce tutti i metadati Yoast automaticamente
**✅ WPML nativo** - usa le funzioni ufficiali WPML per i collegamenti

## Esempio pratico per i tuoi 60 post

**Script di automazione completo:**
```php
<?php
// automation-script.php - da eseguire via WP-CLI

// 1. Leggi i dati esportati dal sito originale
$spanish_posts = json_decode(file_get_contents('exported_spanish_posts.json'), true);

// 2. Inizializza il creatore di traduzioni
$translator = new WPML_Translation_Creator();

// 3. Processa ogni post in batch
foreach ($spanish_posts as $index => $post_data) {
    echo "Processing post {$index}/60: {$post_data['post_title']}\n";
    
    $translator->create_translations($post_data);
    
    // Rate limiting per API di traduzione
    sleep(2);
}

echo "Completed! Run 'wp wpml import process' to finalize connections.\n";
```

**Esecuzione:**
```bash
# 1. Esegui script su sito WPML
wp eval-file automation-script.php

# 2. Finalizza collegamenti traduzioni
wp wpml import process
```

## Timeline di implementazione

- **Giorno 1**: Estrai dati SQL dal sito originale
- **Giorno 2**: Sviluppa script PHP di traduzione e importazione  
- **Giorno 3**: Testa con 5 post
- **Giorno 4**: Esegui migrazione completa dei 60 post
- **Giorno 5**: Verifica e ottimizzazione

Questo approccio ti da controllo totale, è completamente gratuito, e automatizza tutto il processo mantenendo l'integrità dei dati WPML. Vuoi che ti aiuti a sviluppare gli script specifici per il tuo caso?
