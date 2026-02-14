Phase 1: การนำเข้าและเตรียมข้อมูล (Data Ingestion & Preprocessing)

1. Load Data: โหลดไฟล์เอกสาร (PDF)

2. Clean & Tokenize : * นำข้อความมาทำความสะอาด (ลบอักขระพิเศษ, ปรับตัวพิมพ์เล็ก) โดยใช้ไลบรารี PyThaiNLP ในการ "ตัดคำภาษาไทย" และลบ Stopwords ออก เพื่อให้โมเดลเข้าใจขอบเขตของคำไทยได้ถูกต้อง

3. Chunking : นำข้อความที่ผ่านการตัดคำแล้ว มาแบ่ง Chunks โดยใช้ SentenceSplitter ตั้งค่าขนาด chunk_size=700 และchunk_overlap=283

Phase 2: การสร้างฐานข้อมูลเวกเตอร์ (Embedding & Indexing)

4. Embedding: ใช้โมเดล BAAI/bge-m3 (จาก HuggingFace) แปลงข้อความแต่ละ Chunk ให้เป็น Vector

5. Vector Index: นำ Vector ทั้งหมดไปจัดเก็บใน VectorStoreIndex ของ LlamaIndex เพื่อเตรียมพร้อมสำหรับการค้นหา (Retrieval)

Phase 3: กระบวนการดึงข้อมูลและตอบคำถาม (Advanced Retrieval & Generation)

6. Query Transformation: เมื่อผู้ใช้พิมพ์คำถามเข้ามา ระบบจะส่งคำถามไปให้ LLM (ผ่าน Prompt text2text_template) ช่วยสกัดเอาเฉพาะใจความสำคัญ ของคำถามออกมาก่อน เพื่อให้ค้นหาข้อมูลได้แม่นขึ้น

7. Retrieval & Reranking (ดึงและจัดอันดับ)
- ดึงข้อมูล (Retrieve): ค้นหา Chunks ที่มีความหมายคล้ายกับคำถามมากที่สุด 5 อันดับแรก
- จัดอันดับใหม่ (Rerank): ใช้โมเดล BAAI/bge-reranker-v2-m3 มาให้คะแนนความสอดคล้อง (Relevance score) ของทั้ง 5 Chunks นั้นอีกรอบ เพื่อให้ได้บริบทที่แม่นยำที่สุด

8. Synthesize & Refine (ประมวลผลและเกลาคำตอบ)
- ใช้ TreeSummarize สรุปคำตอบเบื้องต้นจากข้อมูลที่ดึงมา
- ส่งคำตอบเบื้องต้นไปให้ LLM ผ่าน refine_template เพื่อทำให้ภาษาอ่านง่าย และบังคับฟอร์แมต เช่น ถ้าเป็นชื่อวิชาต้องเป็นภาษาอังกฤษพร้อมรหัสวิชา และจัดรูปแบบเป็นข้อๆ ให้สวยงาม โดยในโปรเจกต์นี้เลือกใช้โมเดล Llama-4 ผ่าน API ของ Groq เพื่อความรวดเร็ว

Phase 4: การนำไปใช้งาน (Deployment / User Interface)
สร้างหน้าเว็บแอปพลิเคชันอย่างง่ายด้วย Gradio โดยมีช่องให้ผู้ใช้พิมพ์คำถาม (Ask) และแสดงคำตอบ (Answer) ออกมา เพื่อให้พร้อมทดสอบและใช้งานจริง