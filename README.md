#
# dashboard1.py code previous 
import streamlit as st
import psutil
import os
import shutil
import platform
from datetime import datetime
from streamlit_autorefresh import st_autorefresh

# --- CONFIGURATION ---
LOG_DIR = "./logs"
if not os.path.exists(LOG_DIR):
    os.makedirs(LOG_DIR)

st.set_page_config(page_title="Professional Server Monitor", layout="wide", initial_sidebar_state="expanded")

# 1. HEARTBEAT: Auto-refresh every 3 seconds
st_autorefresh(interval=3000, key="datarefresh")

# --- HEADER SECTION ---
col_head1, col_head2 = st.columns([3, 1])
with col_head1:
    st.title("🛡️ Enterprise Server Control Panel")
    st.write(f"Node: `{platform.node()}` | OS: `{platform.system()} {platform.release()}`")
with col_head2:
    boot_time = datetime.fromtimestamp(psutil.boot_time())
    uptime = datetime.now() - boot_time
    st.metric("System Uptime", f"{str(uptime).split('.')[0]}")

st.divider()

# --- SECTION 1: LIVE PERFORMANCE VITALS ---
st.header("📈 Real-Time Performance Vitals")
v1, v2, v3, v4 = st.columns(4)

with v1:
    cpu_usage = psutil.cpu_percent()
    st.metric("CPU Usage", f"{cpu_usage}%")
    st.progress(cpu_usage / 100)

with v2:
    ram = psutil.virtual_memory()
    st.metric("RAM Usage", f"{ram.percent}%", f"{ram.used // (2**20)} MB Used")
    st.progress(ram.percent / 100)

with v3:
    net = psutil.net_io_counters()
    st.metric("Network Sent", f"{net.bytes_sent // (2**20)} MB")
    st.write(f"⬇️ Received: {net.bytes_recv // (2**20)} MB")

with v4:
    load = psutil.getloadavg() if hasattr(psutil, "getloadavg") else [0,0,0]
    st.metric("Load Average (1m)", f"{load[0]}")
    st.write(f"Total Processes: {len(psutil.pids())}")

# --- SECTION 2: DISK STORAGE TOPOLOGY ---
st.header("💽 Storage & Partition Mapping")
partitions = psutil.disk_partitions()
disk_cols = st.columns(len([p for p in partitions if 'cdrom' not in p.opts]))

for i, partition in enumerate(partitions):
    try:
        usage = shutil.disk_usage(partition.mountpoint)
        percent = (usage.used / usage.total) * 100
        with disk_cols[i % len(disk_cols)]:
            st.subheader(f"Mount: {partition.device}")
            st.write(f"Type: `{partition.fstype}`")
            st.progress(percent / 100)
            st.caption(f"{usage.free // (2**30)}GB Free of {usage.total // (2**30)}GB")
    except:
        continue

# --- SECTION 3: LOG MANAGEMENT & AUTOMATION ---
st.divider()
st.header("📂 Log Maintenance & File Security")
files = os.listdir(LOG_DIR)

if files:
    selected_file = st.selectbox("🎯 Target File for Action:", files)
    f_path = os.path.join(LOG_DIR, selected_file)
    
    m_col1, m_col2 = st.columns([2, 1])
    with m_col1:
        f_size = os.path.getsize(f_path) / 1024
        f_time = datetime.fromtimestamp(os.path.getmtime(f_path)).strftime('%Y-%m-%d %H:%M:%S')
        st.code(f"PATH: {os.path.abspath(f_path)}\nSIZE: {f_size:.2f} KB\nLAST MODIFIED: {f_time}")
    
    with m_col2:
        st.write("### Actions")
        if st.button("🗑️ PERMANENT DELETE", type="primary", use_container_width=True):
            os.remove(f_path)
            st.warning(f"ACTION LOGGED: {selected_file} deleted.")
            st.rerun()
else:
    st.info("Log directory is currently empty. Monitoring for new entries...")

# --- FOOTER ---
st.sidebar.markdown("---")
st.sidebar.info(f"**Dashboard Status:** Active\n\n**Refresh Rate:** 3.0s") 
