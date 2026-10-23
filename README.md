# Laravel Semantic Deduper

![Laravel Semantic Deduper vector clusters](art/banner.png)

Group and deduplicate Laravel search or RAG results using cosine similarity of
their embedding vectors. Also includes fuzzy string deduplication and text
chunking utilities.

[![Tests](https://github.com/eloquage/laravel-semantic-deduper/actions/workflows/run-tests.yml/badge.svg)](https://github.com/eloquage/laravel-semantic-deduper/actions/workflows/run-tests.yml)
[![Latest Version](https://img.shields.io/packagist/v/illuma-law/laravel-semantic-deduper)](https://packagist.org/packages/illuma-law/laravel-semantic-deduper)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Requires PHP 8.3+ and Laravel 11, 12, or 13.

## Install

```bash
composer require illuma-law/laravel-semantic-deduper
php artisan vendor:publish --tag=semantic-deduper-config
```

## Cluster results

Pass items with an `embedding` vector and a group key. The clusterer removes
near-duplicates, limits each group, and returns the retained items as DTOs.

```php
use IllumaLaw\SemanticDeduper\SemanticClusterer;

$results = [
    ['id' => 1, 'source' => 'web', 'embedding' => [0.1, 0.2, 0.3]],
    ['id' => 2, 'source' => 'web', 'embedding' => [0.11, 0.19, 0.31]],
    ['id' => 3, 'source' => 'docs', 'embedding' => [0.8, 0.3, -0.2]],
];

$grouped = SemanticClusterer::make()
    ->groupBy('source')
    ->threshold(0.95)
    ->maxPerGroup(3)
    ->cluster($results);

$items = $grouped->allItems();
$ids = $grouped->collectIdentifiers();
```

Defaults for the threshold and result limits are in
`config/semantic-deduper.php`. Use `embeddingKey()` or `idKey()` if your input
uses different field names.

## Utilities

Laravel collections include the `dedupeFuzzy` macro after package discovery:

```php
$unique = collect(['Laravel', 'Laraval', 'Symfony'])
    ->dedupeFuzzy(fn (string $name) => $name, threshold: 85);
```

Split text into overlapping character-based chunks:

```php
use IllumaLaw\SemanticDeduper\Utils\TextChunker;

$chunks = TextChunker::chunk($text, chunkSize: 1200, overlap: 150);
```

## Test

```bash
composer test
composer analyse
vendor/bin/pint --test
```

MIT licensed. See [LICENSE](LICENSE).
