Auto-Rubric-TK Pertiwi - Streamlit deploy package (v10)

Configured for:
- Pink pastel theme (dominant)
- 3 children per report (multi-upload 3 files)
- AI-generated 4 bullet descriptions per photo via Hugging Face (set HUGGINGFACE_TOKEN secret once for public app)
- Output filename: Laporan_Hasil_Karya_Anak.docx

Deployment (Streamlit Cloud):
1. Create a new GitHub repository and push these files.
2. On Streamlit Cloud, create a new app from that repo.
3. In Streamlit Cloud app settings -> Secrets, add HUGGINGFACE_TOKEN with your Hugging Face token (this makes AI calls succeed for all users; set once).
4. Deploy. App URL will be public.
