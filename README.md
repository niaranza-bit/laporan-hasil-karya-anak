import streamlit as st
import requests
from docx import Document
from io import BytesIO
from PIL import Image
import base64

# --- Pengaturan dasar ---
st.set_page_config(page_title="Laporan Hasil Karya Anak TK Pertiwi", page_icon="🎨", layout="wide")

# --- Logo & Identitas ---
col_logo, col_title = st.columns([1, 4])
with col_logo:
    st.image("logo.png", width=100)
with col_title:
    st.markdown("""
    ### **TAMAN KANAK–KANAK PERTIWI 26–68**
    **PRUPUK UTARA KECAMATAN MARGASARI – KABUPATEN TEGAL**  
    Jl. Anggrek No. 01 Rt. 3 Rw. 4 Desa Prupuk Utara, Kec. Margasari, Kab. Tegal
    """, unsafe_allow_html=True)

st.markdown("---")

# --- Input Data Umum ---
col1, col2 = st.columns(2)
with col1:
    kelompok = st.text_input("Kelompok:", "B5")
    semester = st.text_input("Semester:", "I")
with col2:
    tanggal = st.date_input("Hari / Tanggal")
    topik = st.text_input("Topik:", "Aku dan lingkunganku")
    subtopik = st.text_input("Sub Topik:", "Aku sayang tubuhku")

st.markdown("---")

# --- AI Model (baru dan aktif) ---
import requests
import streamlit as st

API_URL = "https://api-inference.huggingface.co/models/google/gemma-2b-it"

headers = {"Authorization": f"Bearer {st.secrets['HUGGINGFACE_TOKEN']}"}

def generate_description(prompt):
    payload = {"inputs": prompt}
    response = requests.post(API_URL, headers=headers, json=payload)

    if response.status_code == 200:
        data = response.json()
        if isinstance(data, list) and "generated_text" in data[0]:
            return data[0]["generated_text"]
        elif isinstance(data, dict) and "generated_text" in data:
            return data["generated_text"]
        else:
            return "Tidak dapat menemukan teks yang dihasilkan."
    else:
        return f"Error {response.status_code}: {response.text}"

# --- Upload Foto ---
st.subheader("📸 Upload Foto Hasil Karya (maks. 3 anak)")
uploaded_files = st.file_uploader("Pilih foto karya anak:", type=["jpg", "jpeg", "png"], accept_multiple_files=True)

if uploaded_files:
    st.markdown("### 📄 Hasil Analisis Otomatis")
    table_data = []

    for i, uploaded_file in enumerate(uploaded_files[:3]):
        name = st.text_input(f"Nama anak ke-{i+1}:", f"Anak {i+1}")
        image = Image.open(uploaded_file)
        st.image(image, width=250)

        with st.spinner("Menganalisis hasil karya..."):
            prompt = f"Buat deskripsi naratif PAUD formal tentang hasil karya {name} berdasarkan foto."
            description = generate_description(prompt)

        st.markdown(f"**Deskripsi {name}:**")
        st.markdown(f"• {description.replace('. ', '.\n• ')}")

        buffered = BytesIO()
        image.save(buffered, format="JPEG")
        img_str = base64.b64encode(buffered.getvalue()).decode()
        table_data.append((name, img_str, description))

    # --- Buat Word ---
    if st.button("📥 Unduh Laporan Word"):
        doc = Document()
        doc.add_heading("LAPORAN HASIL KARYA ANAK TK PERTIWI", level=1)
        doc.add_paragraph(f"Kelompok: {kelompok}     Semester: {semester}")
        doc.add_paragraph(f"Hari / Tanggal: {tanggal}")
        doc.add_paragraph(f"Topik: {topik}     Sub Topik: {subtopik}")
        doc.add_paragraph("")

        for name, img_str, desc in table_data:
            image_data = BytesIO(base64.b64decode(img_str))
            doc.add_picture(image_data, width=None)
            doc.add_paragraph(f"Nama: {name}")
            doc.add_paragraph("Deskripsi foto:")
            for line in desc.split(". "):
                if line.strip():
                    doc.add_paragraph(f"• {line.strip()}", style="List Bullet")
            doc.add_paragraph("")

        buffer = BytesIO()
        doc.save(buffer)
        buffer.seek(0)
        st.download_button(
            label="💾 Simpan ke Word",
            data=buffer,
            file_name="Laporan_Hasil_Karya_TK_Pertiwi.docx",
            mime="application/vnd.openxmlformats-officedocument.wordprocessingml.document"
