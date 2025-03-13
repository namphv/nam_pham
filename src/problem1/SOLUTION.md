# Solution

**Command**

```
grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt | grep -o '"order_id": "[^"]*"' | cut -d'"' -f4 | while read -r order_id; do curl -s "https://example.com/api/$order_id" ; done >> ./output.txt
```
**Explaination**
1. Take all lines with `TSLA` and `sell` string
2. Get `order_id: id` string
3. Split `order_id: id` string to get correct `id`
4. Query to `"https://example.com/api/$order_id"` 1 by 1
5. Append result to output file
