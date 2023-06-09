# Byte\-dictionary encoding<a name="c_Byte_dictionary_encoding"></a>

In byte dictionary encoding, a separate dictionary of unique values is created for each block of column values on disk\. \(An Amazon Redshift disk block occupies 1 MB\.\) The dictionary contains up to 256 one\-byte values that are stored as indexes to the original data values\. If more than 256 values are stored in a single block, the extra values are written into the block in raw, uncompressed form\. The process repeats for each disk block\.

This encoding is very effective on low cardinality string columns\. This encoding is optimal when the data domain of a column is fewer than 256 unique values\.

For columns with the string data type \(CHAR and VARCHAR\) encoded with BYTEDICT, Amazon Redshift performs vectorized scans and predicate evaluations that operate over compressed data directly\. These scans use hardware\-specific single instruction and multiple data \(SIMD\) instructions for parallel processing\. This significantly speeds up the scanning of string columns\. Byte\-dictionary encoding is especially space\-efficient if a CHAR/VARCHAR column holds long character strings\.

Suppose that a table has a COUNTRY column with a CHAR\(30\) data type\. As data is loaded, Amazon Redshift creates the dictionary and populates the COUNTRY column with the index value\. The dictionary contains the indexed unique values, and the table itself contains only the one\-byte subscripts of the corresponding values\.

**Note**  
Trailing blanks are stored for fixed\-length character columns\. Therefore, in a CHAR\(30\) column, every compressed value saves 29 bytes of storage when you use the byte\-dictionary encoding\.

The following table represents the dictionary for the COUNTRY column\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/redshift/latest/dg/c_Byte_dictionary_encoding.html)

The following table represents the values in the COUNTRY column\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/redshift/latest/dg/c_Byte_dictionary_encoding.html)

The total compressed size in this example is calculated as follows: 6 different entries are stored in the dictionary \(6 \* 30 = 180\), and the table contains 10 1\-byte compressed values, for a total of 190 bytes\.