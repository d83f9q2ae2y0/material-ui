<?php

namespace App\Service;

use PhpOffice\PhpSpreadsheet\IOFactory;

class ExcelProcessor
{
    public function processExcelFile(string $filePath, int $minItemsRequired = 1): array
    {
        // Load the Excel file
        $spreadsheet = IOFactory::load($filePath);
        $worksheet = $spreadsheet->getActiveSheet();
        
        // Get all rows
        $rows = $worksheet->toArray();
        
        // Skip header row if exists (adjust as needed)
        $dataRows = array_slice($rows, 1);
        
        // Group data by column A (id)
        $grouped = [];
        
        foreach ($dataRows as $row) {
            $idA = $row[0] ?? null; // Column A
            $valueB = $row[1] ?? null; // Column B
            $valueC = $row[2] ?? null; // Column C
            
            // Skip empty rows
            if (empty($idA)) {
                continue;
            }
            
            // Initialize array for this ID if not exists
            if (!isset($grouped[$idA])) {
                $grouped[$idA] = [
                    'id' => $idA,
                    'ddd' => []
                ];
            }
            
            // Add B and C values to the ddd array
            $grouped[$idA]['ddd'][] = [
                'b' => $valueB,
                'c' => $valueC
            ];
        }
        
        // Convert to indexed array
        $result = array_values($grouped);
        
        // Filter based on criteria (number of items in "ddd" array)
        $result = array_filter($result, function($item) use ($minItemsRequired) {
            return count($item['ddd']) >= $minItemsRequired;
        });
        
        // Re-index array after filtering
        return array_values($result);
    }
    
    /**
     * Alternative method if you want more control over filtering
     */
    public function processExcelFileWithCustomFilter(
        string $filePath, 
        callable $filterCallback
    ): array {
        $spreadsheet = IOFactory::load($filePath);
        $worksheet = $spreadsheet->getActiveSheet();
        $rows = $worksheet->toArray();
        $dataRows = array_slice($rows, 1);
        
        $grouped = [];
        
        foreach ($dataRows as $row) {
            $idA = $row[0] ?? null;
            $valueB = $row[1] ?? null;
            $valueC = $row[2] ?? null;
            
            if (empty($idA)) {
                continue;
            }
            
            if (!isset($grouped[$idA])) {
                $grouped[$idA] = [
                    'id' => $idA,
                    'ddd' => []
                ];
            }
            
            $grouped[$idA]['ddd'][] = [
                'b' => $valueB,
                'c' => $valueC
            ];
        }
        
        $result = array_values($grouped);
        $result = array_filter($result, $filterCallback);
        
        return array_values($result);
    }
}

// Example usage in a Controller:
/*
namespace App\Controller;

use App\Service\ExcelProcessor;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;

class ExcelController extends AbstractController
{
    public function processExcel(ExcelProcessor $processor): Response
    {
        $filePath = '/path/to/your/file.xlsx';
        
        // Basic usage: filter items with at least 3 entries
        $result = $processor->processExcelFile($filePath, 3);
        
        // Or with custom filter callback
        $result = $processor->processExcelFileWithCustomFilter(
            $filePath, 
            function($item) {
                // Keep only items with exactly 5 entries in ddd array
                return count($item['ddd']) === 5;
            }
        );
        
        // Process the result
        foreach ($result as $item) {
            echo "ID: " . $item['id'] . "\n";
            echo "Number of entries: " . count($item['ddd']) . "\n";
            
            foreach ($item['ddd'] as $entry) {
                echo "  B: " . $entry['b'] . ", C: " . $entry['c'] . "\n";
            }
        }
        
        return new Response('Processed');
    }
}
*/
