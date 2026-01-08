# video-to-prompt-tool
video
import streamlit as st
import cv2
import tempfile
import google.generativeai as genai
from PIL import Image
import os
import time

# 页面配置
st.set_page_config(page_title="AI 分镜反推工具", layout="wide", page_icon="🎬")

# 自定义CSS让界面更像专业SaaS工具
st.markdown("""
<style>
    .stButton>button {width: 100%; border-radius: 5px; height: 3em; background-color: #FF4B4B; color: white;}
    .reportview-container {background: #f0f2f6;}
</style>
""", unsafe_allow_html=True)

# 核心函数：提取关键帧
def extract_frames(video_path, interval_sec=2):
    cap = cv2.VideoCapture(video_path)
    fps = cap.get(cv2.CAP_PROP_FPS)
    frames = []
    timestamps = []
    
    current_frame = 0
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret: break
        
        if int(current_frame % (int(fps) * interval_sec)) == 0:
            frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            frames.append(Image.fromarray(frame_rgb))
            timestamps.append(current_frame / fps)
            
        current_frame += 1
    cap.release()
    return frames, timestamps

# 核心函数：AI分析
def analyze_image(image, api_key):
    genai.configure(api_key=api_key)
    # 使用 Flash 模型，速度快且免费额度高
    model = genai.GenerativeModel('gemini-1.5-flash') 
    
    prompt = """
    Analyze this movie frame. Provide a concise prompting description for AI image generation (like Midjourney).
    Format: [Subject/Action], [Camera Angle/Shot Type], [Lighting/Atmosphere], [Art Style/Texture].
    Keep it strictly descriptive, comma-separated.
    """
    try:
        response = model.generate_content([prompt, image])
        return response.text
    except Exception as e:
        return "Error analyzing frame."

# --- 侧边栏 ---
with st.sidebar:
    st.title("🛠️ 工具设置")
    # 可以在这里预埋你的Key，或者让用户输入
    api_key = st.text_input("Gemini API Key", type="password", help="如果没有，去 Google AI Studio 申请免费版")
    interval = st.slider("抽帧间隔 (秒)", 1, 10, 3)
    st.markdown("---")
    st.markdown("**关于工具**\n\n此工具用于快速拆解视频分镜，并反推 AIGC 提示词。")

# --- 主界面 ---
st.title("🎬 视频智能分镜 & Prompt 反推")

uploaded_file = st.file_uploader("拖入参考视频 (MP4/MOV)", type=["mp4", "mov"])

if uploaded_file and st.button("开始智能分析"):
    if not api_key:
        st.warning("请先在左侧输入 API Key")
        st.stop()

    # 临时保存视频以供 OpenCV 读取
    tfile = tempfile.NamedTemporaryFile(delete=False) 
    tfile.write(uploaded_file.read())
    
    with st.spinner('正在拆解分镜...'):
        frames, timestamps = extract_frames(tfile.name, interval)
    
    st.success(f"拆解完成，共 {len(frames)} 个关键帧。正在调用 AI 分析...")
    
    # 进度条
    progress_bar = st.progress(0)
    
    # 结果展示区
    for i, (frame, ts) in enumerate(zip(frames, timestamps)):
        with st.container():
            col1, col2 = st.columns([1, 2])
            with col1:
                st.image(frame, use_column_width=True, caption=f"⏱️ {ts:.1f}s")
            with col2:
                result = analyze_image(frame, api_key)
                st.markdown(f"**Keyframe #{i+1} Prompt:**")
                st.code(result, language="text")
        
        st.divider()
        progress_bar.progress((i + 1) / len(frames))
    
    os.remove(tfile.name) # 清理垃圾
    streamlit
opencv-python-headless
google-generativeai
Pillow
numpy
