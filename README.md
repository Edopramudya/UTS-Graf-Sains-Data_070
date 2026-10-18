# UTS-Graf-Sains-Data
Proyek ini membangun sistem rekomendasi item berbasis graf pada dataset e-commerce RetailRocket (Roman Zykov, Kaggle). Data interaksi pengunjung dimodelkan sebagai graf bipartit di Neo4j, diproyeksikan ke graf item-item, dihitung kemiripannya (cosine similarity), lalu dipakai untuk rekomendasi KNN dan dievaluasi dengan Hit@5 dan NDCG@5.
