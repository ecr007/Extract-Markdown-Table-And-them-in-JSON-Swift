# Extract-Markdown-Table-And-Then-in-JSON-Swift

To detect and extract a Markdown table from a string and convert it to JSON in the specified format, you can use regular expressions and some parsing logic. Here's an example of how you can achieve this:

```swift
import Foundation

func extractMarkdownTable(_ input: String) -> [String: Any]? {
    // Regex pattern to match Markdown table rows
    let rowRegexPattern = "\\|(.*)\\|"
    let rowRegex = try! NSRegularExpression(pattern: rowRegexPattern, options: [])
    
    // Regex pattern to match table header
    let headerRegexPattern = "\\|.*?\\|"
    let headerRegex = try! NSRegularExpression(pattern: headerRegexPattern, options: [])
    
    // Find table rows
    let rowMatches = rowRegex.matches(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count))
    guard !rowMatches.isEmpty else {
        return nil
    }
    
    // Extract header
    let headerMatch = headerRegex.firstMatch(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count))
    guard let headerRange = headerMatch?.range else {
        return nil
    }
    let headerString = (input as NSString).substring(with: headerRange)
    let headerComponents = headerString.components(separatedBy: "|").map { $0.trimmingCharacters(in: .whitespaces) }.filter { !$0.isEmpty }
    
    // Extract table rows
    var rows: [[String]] = []
    for rowMatch in rowMatches {
        let rowRange = rowMatch.range
        let rowString = (input as NSString).substring(with: rowRange)
        let rowComponents = rowString.components(separatedBy: "|").map { $0.trimmingCharacters(in: .whitespaces) }.filter { !$0.isEmpty }
        rows.append(rowComponents)
    }
    
    // Create JSON dictionary
    let json: [String: Any] = [
        "header": headerComponents,
        "body": rows
    ]
    
    return json
}
```

In this code, we use regular expressions to match and extract the header and rows of the Markdown table. The rowRegexPattern matches the table rows, and the headerRegexPattern matches the table header. We then extract the relevant components and construct a JSON dictionary with the specified format.

Here's an example usage of the extractMarkdownTable function:

```swift
let markdownString = """
| title 1 | title 2 | title 3 |
| ------- | ------- | ------- |
| row 1 value 1 | row 1 value 2 | row 1 value 3 |
| row 2 value 1 | row 2 value 2 | row 2 value 3 |
"""

if let json = extractMarkdownTable(markdownString) {
    do {
        let jsonData = try JSONSerialization.data(withJSONObject: json, options: .prettyPrinted)
        if let jsonString = String(data: jsonData, encoding: .utf8) {
            print(jsonString)
        }
    } catch {
        print("Failed to convert to JSON: \(error)")
    }
} else {
    print("No Markdown table found.")
}
```
