# Logstash 配置文件說明

## JDBC 輸入設定

### jdbc_driver_library
`jdbc_driver_library` 指定了 JDBC 驅動程式的路徑，這需要與你的資料庫相對應。

### jdbc_driver_class
`jdbc_driver_class` 是 JDBC 驅動程式的類別名稱。這個需要根據你的資料庫來設定。

### jdbc_connection_string
`jdbc_connection_string` 是資料庫的連接字符串。它包括了資料庫的 URL 以及可能的其他設定。

### jdbc_user
`jdbc_user` 是資料庫的使用者名稱。

### jdbc_password
`jdbc_password` 是資料庫的使用者密碼。

### schedule
`schedule` 是一個 cron 表達式，用來設定 Logstash 執行資料查詢的頻率。

### add_field
`add_field` 參數可以添加一個或多個欄位到事件。每個添加的欄位都是一個名/值對。

### statement_filepath
`statement_filepath` 是一個包含 SQL 查詢的檔案的路徑。

### use_column_value
`use_column_value` 參數決定了 Logstash 如何記錄查詢的最後運行時間。如果將 use_column_value 設為 true，Logstash 會在 tracking_column 指定的列中找到最大的值，並在下次查詢時使用該值。這表示每次查詢都會從上次運行的結果之後的數據開始。
例如，如果 tracking_column 設為 "createtime"，那麼 Logstash 會在每次運行查詢時找到 "createtime" 列的最大值，並在下次查詢時從該值之後的數據開始。這對於處理增量數據（即新添加的行）非常有用，因為它可以確保每次運行查詢時都只處理新數據。

### tracking_column
`tracking_column` 是用於追蹤查詢最後運行時間的欄位名稱。

### tracking_column_type
`tracking_column_type` 是追蹤欄位的類型。這個值可以是 "numeric" 或 "timestamp"。

### last_run_metadata_path
`last_run_metadata_path` 是用於儲存查詢最後運行時間的檔案路徑。

### jdbc_paging_enabled
`jdbc_paging_enabled` 參數可以開啟或關閉 JDBC 分頁功能。如果設為 true，Logstash 會使用 SQL 的 LIMIT 和 OFFSET 語句來進行分頁查詢。

## Elasticsearch 輸出設定

### hosts
`hosts` 是一個包含 Elasticsearch 服務地址的陣列。

### index
`index` 是要寫入的 Elasticsearch 索引名稱。

### document_id
`document_id` 是寫入 Elasticsearch 的文檔的唯一標識符。

### ssl
`ssl` 參數可以開啟或關閉 SSL 連接到 Elasticsearch。

## 條件式輸出

在 Logstash 的配置文件中，你可以使用 if 條件來決定特定事件的輸出位置。在這個配置中，我們根據 `database_name` 欄位的值來判定要將數據寫入哪個 Elasticsearch 索引。如果 `database_name` 的值為 "m_otc"，則將數據寫入 "m_otc_server_ap_log" 索引，如果值為 "qat_m_otc"，則將數據寫入 "qat_m_otc_server_ap_log" 索引。

## 環境變數

這個配置文件使用了一些環境變數，如 `MYSQL_CONNECTION_STRING`, `DB_ONE_NAME`, `DB_ONE_USER`, `DB_ONE_PASSWORD` 等。這些變數的值需要在運行 Logstash 的環境中事先定義好。使用環境變數的好處是可以將敏感或變動性大的信息（如密碼、主機名稱等）從配置文件中分離出來，以增加靈活性和安全性。

# useCursorFetch=true
useCursorFetch=true 是 MySQL JDBC 驅動器的一個選項，用於啟用游標取回模式。在這種模式下，當執行 SQL 查詢後，驅動器並不會一次性地將所有查詢結果從資料庫伺服器取回到客戶端，而是將結果保持在伺服器端，並使用游標來一次取回部分結果。這種模式對於處理大量數據的查詢非常有用，因為它可以避免一次性將大量數據載入到記憶體中，從而降低了記憶體壓力。
如果從 jdbc_connection_string 中移除 useCursorFetch=true，則 JDBC 驅動器將切換到預設的取回模式，即一次性將所有查詢結果從資料庫伺服器取回到客戶端。對於處理小型數據集的查詢，這種模式通常可以正常工作。然而，如果你的 SQL 查詢返回了大量的數據，這可能會導致記憶體不足的問題，尤其是當 Logstash 或其它應用的記憶體資源有限時。此外，如果你的查詢經常返回大量數據，那麼使用預設的取回模式可能會導致資料庫伺服器的壓力增大，因為它需要一次性將所有數據送到客戶端。