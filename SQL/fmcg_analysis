use portofolio;

-- VIEW: vw_enriched_sales 
-- Tujuan
-- Tabel fakta utama yang menggabungkan data transaksi, pembersihan,
--   kalkulasi metrik bisnis, dan join dengan tabel dimensi.

-- Metrics: Gross Sales, Net Sales, COGS, Gross Profit
-- Dimensi: Store info (country, city, channel), Product info, Supplier info

DROP VIEW IF EXISTS vw_enriched_sales;

Create view vw_enriched_sales AS
-- Tahap 1: Pembersihan
-- Trim digunakan untuk menghapus spasi 
-- Mengubah Null menjadi 0 menggunakan COALESCE
-- Mengubah Format tanggal menjadi Date
-- Mengubah Promo Flag dari angka menjadi label Promo / Non-Promo
With cleaned_raw as(
 Select 
  Date(date) as Date,
  Trim(Store_id) as Store_id,
     Trim(sku_id) as Sku_id,
     Trim(supplier_id) as Supplier_id,
     Coalesce(units_sold, 0) as Unit_sold,
     Coalesce(list_price, 0) as List_price,
     Coalesce(stock_on_hand, 0) as Stock,
     Coalesce(purchase_cost,0) as purchase_cost,
     Coalesce(discount_pct) as discount_pct,
     Case
   When promo_flag = 1 Then 'Promo'
   When promo_flag = 0 Then 'Non-Promo'
   else 'unknown'
     End as Promo_flag,
     lead_time_days
 From fmcg_raw),
 
 -- Tahap 2 business matrix
 -- Gross sales = Unit Sold X List Price (Sebelum Diskon)
 -- Net Sales = (Unit Sold  X List Price (1 - Discount) (Setelah Discount)
 -- COGS = Purchase Cost X Unit Sold
 -- Penggunaan Round untuk membulatkan angka menjadi 2 angka setelah titik (.)
calculated_sales AS(
	Select 
		Date,
		Store_id,
		Sku_id,
		Supplier_id,
		Unit_sold,
		List_price,
		Stock,
		discount_pct,
		Promo_flag,
		lead_time_days,
		round(Unit_sold * List_Price,2) as Gross_sales,
        Round((Unit_sold * List_Price * (1 - discount_pct)), 2) AS Net_sales,
        round(purchase_cost * unit_sold, 2) as COGS
	From cleaned_raw)
   
-- Tahap 3 Join
-- product_masterr: Info produk (SKU_Name, Category, Brand)
-- store_master: Info toko (Country, City, Channel)
-- supplier_master: Info supplier
    Select
		cs.Date,
        cs.Store_id,
        sm.country,
        sm.city,
        sm.channel,
        
        cs.Sku_id,
        coalesce(pm.Sku_Name, 'Unknown') AS SKU_Name,
        coalesce(pm.Category, 'Unknown') AS Category,
        coalesce(pm.Brand, 'Unknown') AS Brand,
        
        cs.Supplier_id,
        sp.supplier_name,
        Promo_flag,
        stock,
        
        cs.Unit_sold,
        cs.list_price,
        cs.Gross_sales,
        cs.Net_sales,
        cs.COGS,
        Round((cs.Net_sales - cs.COGS),2) as Gross_profit,
        cs.discount_pct
	
    From calculated_sales as cs
    Left join product_masterr as pm
		on cs.Sku_id = pm.sku_id
	Left Join store_master as sm
		on cs.store_id = sm.store_id
	Left Join supplier_master as sp
		on cs.Supplier_id = sp.supplier_id;

-- VERIFIKASI DATA
Select * 
From vw_enriched_sales;


-- VIEW vw_daily_store_fact
-- Tujuan: Aggregasi metrix per hari, toko, dan Kategori
-- Perencanaan inventory harian
-- Tracking target sales harian
DROP VIEW IF EXISTS vw_daily_store_fact;

Create view vw_daily_store_fact AS
	select 
		Date,
        Store_id,
        Category,
        
        sum(Unit_sold) as Total_Unit_Sold,
        sum(gross_sales) as Total_Gross_Sales,
        sum(Net_Sales) as Total_Net_Sales,
        sum(COGS) as Total_COGS,
        sum(Gross_profit) as Total_Gross_Profit,
-- GPM: Metrix Profitabilitas per Kategori
        ROUND(Sum(Gross_Profit) / Sum(Net_Sales) * 100,2) AS GPM
        
	From vw_enriched_sales
    Group by Date,Store_id,category;
-- Verifikasi Data
Select *
From vw_enriched_sales;
       
-- VIEW vw_summary_channel
-- TUJUAN: 
-- Agregasi metrik per Tanggal, Kota, Channel, dan Kategori
-- Breakdown sales per channel dan kota
-- Perbandingan GPM antar channel
 DROP VIEW IF EXISTS vw_summary_channel;
 
Create view vw_summary_channel AS
	select 
		Date,
        city,
        channel,
        Category,
        
        sum(Unit_sold) as Total_Unit_Sold,
        sum(gross_sales) as Total_Gross_Sales,
        sum(Net_Sales) as Total_Net_Sales,
        sum(COGS) as Total_COGS,
        sum(Gross_profit) as Total_Gross_Profit,
        ROUND(Sum(Gross_Profit) / Sum(Net_Sales) * 100,2) AS GPM
        
	From vw_enriched_sales
    Group by Date,city,channel,category
    order by Date desc;
    
-- Verifikasi Data
select *
From vw_summary_channel;


-- VIEW vw_rfm_analysis
-- Tujuan: Melakukan Segmentasi Performa Channel
-- 		R (Recency): Jumlah hari sejak pesanan terakhir (Indikator keaktifan operasional toko).
-- 		F (Frequency): Seberapa rutin toko melakukan restock (Indikator stabilitas permintaan).
-- 		M (Monetary): Total nilai Net Sales (Indikator kontribusi pendapatan terhadap perusahaan).
-- LOGIKA SKORING:
-- 		Menggunakan NTILE(4) untuk membagi toko ke dalam kuartil (Skor 1-4).
-- 		Skor 4: Mewakili 25% kelompok toko dengan performa tertinggi di tiap metrik.
-- OUTPUT:
-- 		R_Score, F_Score, M_Score: Skor individu per metrik.
-- 		Total_RFM_Score: Akumulasi skor (Range 3 - 12) untuk penentuan prioritas.
-- 		Semakin tinggi akumulasi skor, semakin tinggi prioritasnya

DROP VIEW IF EXISTS vw_rfm_analysis;

CREATE VIEW vw_rfm_analysis AS
-- Tahap 1 Base Calculation
-- Recency diambil dari last transactions per store
-- Frequency diambil dari jumlah hari dengan transaksi
-- Monetary diambil dari Total Spending
WITH rfm_base AS(
	Select  
		Store_id,
        max(Date) as Last_Transactions,
        Count(distinct Date) as Frequency,
        Sum(Net_Sales) as Monetary
	From vw_enriched_sales
    Group by Store_id
),
-- Tahap 2 scoring NTILE(4)
-- 4 = paling baik (paling baru, paling sering dan paling banyak spending
-- 1 = paling buruk
rfm_score AS(
	Select *,
    NTILE(4) Over (Order by Last_Transactions ASC) AS R_Score,
    NTILE(4) Over (Order by Frequency ASC) AS F_Score,
    NTILE(4) Over (Order by Monetary ASC) AS M_Score
    From rfm_base
)
Select 
	*,
-- Total Score dari 3 - 12 (semua score tinggi merupakan hasil terbaik)
    (R_Score + F_Score + M_Score) AS Total_RFM_Score,
  -- Segmentasi B2B  
   CASE 
    -- 1. Champions: Skor tertinggi di semua lini (Prioritas Utama)
    WHEN R_Score = 4 AND F_Score = 4 AND M_Score = 4 THEN 'Champions'
    
    -- 2. Loyal Customers: Skor cukup tinggi, rutin belanja
    WHEN R_Score >= 3 AND F_Score >= 3 THEN 'Loyal Customers'
    
    -- 3. At Risk: Dulu sering belanja (F tinggi) tapi sekarang sudah lama tidak (R rendah)
    WHEN R_Score <= 2 AND F_Score >= 3 THEN 'At Risk (Churn Warning)'
    
    -- 4. High Value Lost: Jarang belanja tapi sekalinya belanja nilainya besar (M tinggi)
    WHEN R_Score <= 2 AND M_Score >= 3 THEN 'High Value Lost'
    
    -- 5. New / Potential: Baru bertransaksi tapi frekuensi belum tinggi
    WHEN R_Score >= 3 AND F_Score <= 2 THEN 'Potential'
    
    -- 6. Lost: Skor rendah di semua lini
    WHEN R_Score <= 2 AND F_Score <= 2 THEN 'Lost'

    ELSE 'Others'
END AS Customer_Segment
From rfm_score;

-- Verifikasi Data
Select * 
From vw_rfm_analysis;

-- VIEW vw_abc_xyz_analysis
-- TUJUAN: 
-- Klasifikasi produk berdasarkan Profit (ABC) dan Stabilitas (XYZ)

-- ABC Classification (Berdasarkan Pareto Principle):
-- 	A: 80% profit (prioritas tinggi)
-- 	B: 15% profit (prioritas sedang)
-- 	C: 5% profit (prioritas rendah)
--
--   XYZ Classification (Berdasarkan Volatilitas Permintaan):
-- 	X: Stabil (Coefficient of Variance < 0.5)
-- 	Y: Fluktuatif (CV 0.5 - 1.0)
-- 	Z: Tidak Stabil (CV > 1.0)

-- 	AX: Best sellers yang stabil → Always stock
-- 	CZ: Slow movers yang tidak stabil → Consider discontinuation
CREATE VIEW vw_abc_xyz_analysis AS
WITH monthly_sales AS (
    -- Tahap 1: Aggregasi bulanan untuk menghilangkan 'noise' data harian
    SELECT
        Sku_id,
        SKU_Name,
        Category,
        Channel,
        DATE_FORMAT(Date, '%Y-%m') AS YearMonth,
        SUM(Unit_sold) AS Monthly_Unit_Sold,
        SUM(Gross_Profit) AS Monthly_Gross_Profit
    FROM vw_enriched_sales
    GROUP BY 
        Sku_id, SKU_Name, Category, Channel, YearMonth
),

sales_stats AS (
    -- Tahap 2: Kalkulasi statistik dasar untuk Profit dan Volatilitas
    SELECT
        Sku_id,
        SKU_Name,
        Category,
        Channel,
        SUM(Monthly_Gross_Profit) AS Total_Gross_Profit,
        SUM(Monthly_Unit_Sold) AS Total_Unit_Sold,
        AVG(Monthly_Unit_Sold) AS Avg_Monthly_Unit,
        STDDEV(Monthly_Unit_Sold) AS Sales_Volatility
    FROM monthly_sales
    GROUP BY 
        Sku_id, SKU_Name, Category, Channel
),

abc_logic AS (
    -- Tahap 3: Menghitung persentase kumulatif profit (Pareto Analysis)
    SELECT
        *,
        SUM(Total_Gross_Profit) OVER (
            ORDER BY Total_Gross_Profit DESC
        ) / 
        SUM(Total_Gross_Profit) OVER () 
        AS Cumulative_GProfit_PCT
    FROM sales_stats
)

	-- Tahap 4: Final Klasifikasi menggunakan Coef_Var (Coefficient of Variation)
SELECT
    Sku_id,
    SKU_Name,
    Category,
    Channel,
    Total_Gross_Profit,
    Total_Unit_Sold,
    Sales_Volatility,
    -- Coefficient of Variation (Coef_Var) = StdDev / Mean
    ROUND(Sales_Volatility / NULLIF(Avg_Monthly_Unit, 0),2) AS Coef_Var,

    -- ABC: Berdasarkan kontribusi profit kumulatif
    CASE
        WHEN Cumulative_GProfit_PCT <= 0.80 THEN 'A' -- Top 80% Gross Profit
        WHEN Cumulative_GProfit_PCT <= 0.95 THEN 'B' -- 15% Gross Profit
        ELSE 'C' -- 5% Gross Profit
    END AS ABC_CLASS,

    -- XYZ: Berdasarkan stabilitas permintaan (Coefficient of Variation)
    CASE
        WHEN Sales_Volatility / NULLIF(Avg_Monthly_Unit, 0) < 0.5 THEN 'X' -- Stabil
        WHEN Sales_Volatility / NULLIF(Avg_Monthly_Unit, 0) < 1.0 THEN 'Y' -- Fluktuatif
        ELSE 'Z' -- Tidak Stabil
    END AS XYZ_CLASS

FROM abc_logic
ORDER BY Total_Gross_Profit DESC;

Select * 
From vw_abc_xyz_analysis;

Select Count(*)
From product_master;

Select Count(*)
From product_masterr;

SELECT gross_sales, net_sales FROM vw_enriched_sales ;
Select * from vw_enriched_sales;


-- Cek langsung dari fmcg_raw (tabel asli)
SELECT 
    date,
    units_sold,
    list_price,
    discount_pct,
    (units_sold * list_price) AS calculated_gross,
    (units_sold * list_price * (1 - discount_pct)) AS calculated_net
FROM fmcg_raw
LIMIT 10;




