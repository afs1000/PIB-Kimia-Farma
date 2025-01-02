SELECT 
    tt.transaction_id,
    tt.date AS date,
    tt.branch_id,
    kc.branch_name,
    kc.kota,
    kc.provinsi,
    COALESCE(tt.rating, kc.rating) AS rating,
    tt.customer_name,
    tt.product_id,
    p.product_name,
    p.price,
    tt.discount_percentage,
    
    ## Perhitungan persentase gross laba berdasarkan harga
    CASE 
        WHEN p.price <= 50000 THEN 10
        WHEN p.price > 50000 AND p.price <= 100000 THEN 15
        WHEN p.price > 100000 AND p.price <= 300000 THEN 20
        WHEN p.price > 300000 AND p.price <= 500000 THEN 25
        ELSE 30
    END AS persentase_gross_laba,
    
    ## Perhitungan nett_sales setelah diskon
    p.price * (1 - tt.discount_percentage / 100) AS nett_sales,
    
    ## Perhitungan nett_profit berdasarkan laba dan nett_sales
    (p.price * (1 - tt.discount_percentage / 100)) * 
    CASE 
        WHEN p.price <= 50000 THEN 0.10
        WHEN p.price > 50000 AND p.price <= 100000 THEN 0.15
        WHEN p.price > 100000 AND p.price <= 300000 THEN 0.20
        WHEN p.price > 300000 AND p.price <= 500000 THEN 0.25
        ELSE 0.30
    END AS nett_profit,
    
    ## Rating transaksi dihitung dari rata-rata rating dan persentase diskon
    (COALESCE(tt.rating, kc.rating) + (100 - tt.discount_percentage)) / 2 AS rating_transaksi
FROM 
    `Kimia_Farma.Tabel Transaksi` tt
LEFT JOIN 
    `Kimia_Farma.kantor cabang` kc
ON 
    tt.branch_id = kc.branch_id
LEFT JOIN 
    `Kimia_Farma.produk` p
ON 
    tt.product_id = p.product_id
LEFT JOIN 
    `Kimia_Farma.Inventori` inv
ON 
    tt.branch_id = inv.branch_id
    AND tt.product_id = inv.product_id
WHERE 
    tt.date IS NOT NULL
ORDER BY 
    tt.date ASC;
