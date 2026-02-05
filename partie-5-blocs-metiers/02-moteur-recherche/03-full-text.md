# 21.3 Full-text

## Requêtes de recherche

```rust
use tantivy::query::{QueryParser, BooleanQuery, Occur};

impl SearchEngine {
    pub fn search_advanced(&self, query: &str, filters: &SearchFilters) -> Result<Vec<SearchResult>> {
        let reader = self.index.reader()?;
        let searcher = reader.searcher();
        
        let mut query_parser = QueryParser::for_index(
            &self.index,
            vec![self.title_field, self.body_field],
        );
        
        // Boost du titre (plus important que le contenu)
        query_parser.set_field_boost(self.title_field, 2.0);
        
        let text_query = query_parser.parse_query(query)?;
        
        // Ajouter des filtres
        let mut queries = vec![(Occur::Must, text_query)];
        
        if let Some(date_filter) = &filters.date_range {
            let date_query = self.build_date_query(date_filter)?;
            queries.push((Occur::Must, date_query));
        }
        
        let boolean_query = BooleanQuery::new(queries);
        let top_docs = searcher.search(&boolean_query, &TopDocs::with_limit(20))?;
        
        // Convertir en résultats...
        Ok(results)
    }
}

pub struct SearchFilters {
    pub date_range: Option<(DateTime<Utc>, DateTime<Utc>)>,
    pub category: Option<String>,
}
```

## Scoring et ranking

```rust
use tantivy::collector::TopDocs;
use tantivy::query::Query;

impl SearchEngine {
    pub fn search_with_custom_scoring(&self, query: &str) -> Result<Vec<SearchResult>> {
        let reader = self.index.reader()?;
        let searcher = reader.searcher();
        
        let query_parser = QueryParser::for_index(
            &self.index,
            vec![self.title_field, self.body_field],
        );
        
        let query = query_parser.parse_query(query)?;
        
        // TopDocs avec scoring personnalisé
        let top_docs = searcher.search(&query, &TopDocs::with_limit(20))?;
        
        // Les résultats sont déjà triés par score décroissant
        Ok(top_docs.into_iter()
            .map(|(score, doc_address)| {
                // Récupérer le document et construire SearchResult
                // ...
            })
            .collect())
    }
}
```

## Filtres et facettes

```rust
use tantivy::query::{TermQuery, Term};
use tantivy::schema::Field;

impl SearchEngine {
    pub fn search_with_facets(&self, query: &str, category: Option<&str>) -> Result<Vec<SearchResult>> {
        let reader = self.index.reader()?;
        let searcher = reader.searcher();
        
        let query_parser = QueryParser::for_index(
            &self.index,
            vec![self.title_field, self.body_field],
        );
        
        let text_query = query_parser.parse_query(query)?;
        
        // Ajouter un filtre de catégorie si spécifié
        if let Some(cat) = category {
            let category_field = self.schema.get_field("category")?;
            let term = Term::from_field_text(category_field, cat);
            let category_query = Box::new(TermQuery::new(term, IndexRecordOption::Basic));
            
            let boolean_query = BooleanQuery::new(vec![
                (Occur::Must, text_query),
                (Occur::Must, category_query),
            ]);
            
            let top_docs = searcher.search(&boolean_query, &TopDocs::with_limit(20))?;
            // ...
        } else {
            let top_docs = searcher.search(&text_query, &TopDocs::with_limit(20))?;
            // ...
        }
        
        Ok(results)
    }
}
```

## Résumé

- **Requêtes avancées** : Combinaison de plusieurs critères
- **Scoring** : Ranking automatique par pertinence
- **Filtres** : Restreindre par catégorie, date, etc.
- **Facettes** : Navigation par facettes (catégories, dates)
