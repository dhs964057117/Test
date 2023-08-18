# Test
测试


#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os, sys
import win32file
import ctypes
import subprocess
import gradio as gr
import random
from PIL import Image
from IPython.display import Image as IPImage
from pathlib import Path
import shutil
from gradio import networking
import glob
import json
from flask import request
import requests
def main():
    # 打开文件
    path = r"C:/Users/philip.du/Desktop/test/10.md"
    fd = os.open( path, os.O_RDWR|os.O_CREAT )

    # 关闭文件
    os.close(fd)

    # 创建以上文件的拷贝
    dst = r"F:/AI_Project/10.md"
    os.symlink( path, dst)

    print ("创建硬链接成功!!")

def test():
    try:
        win32file.CreateSymbolicLink(r"C:/Users/philip.du/Desktop/test/10.md",r"F:/AI_Project/10.md")
        print("创建软链接成功!!")
    except Exception as e:
        print (e)

#删除文件
def delete_file(path):
    if os.path.exists(path):
        os.remove(path)

#判断路径下是否有其他文件
def path_has_other_file(path):
    if os.path.exists(path):
        if os.path.isdir(path):
            for root, dirs, files in os.walk(path):
                if len(files) > 0:
                    return True
                else:
                    return False
        else:
            return True
    else:
        return False

def create_junction(src, dst):
    # if not os.path.isdir(src):
    #     raise NotADirectoryError(f"Source directory '{src}' does not exist.")
    
    # if os.path.exists(dst) and not os.path.islink(dst):
    #         print(f"junction node has '{dst}' already exists")
    #         return
    # if os.path.exists(dst) or os.path.islink(dst):
    #     if os.path.exists(dst) and os.path.islink(dst):
    #         print(f"junction node has '{dst}' already exists")
    #         return
    #     if os.path.exists(dst) and not os.path.islink(dst) and path_has_other_file(dst):
    #         print(f"Destination '{dst}' already exists. please delete it first")
    #         return
        # raise FileExistsError(f"Destination '{dst}' already exists.")


    with open(os.devnull, 'w') as devnull:
        try:
            subprocess.call('cmd.exe /c mklink /J "%s" "%s"' % (dst, src), shell=True, stdout=devnull, stderr=devnull)
            print("创建目录链接成功!!")
        except Exception as e:
            print (e)

# 指定源目录和Junction名称
source_directory = 'path/to/source_directory'
junction_name = 'path/to/junction_name'

# 创建Junction
# create_junction(source_directory, junction_name)

def test_sysmbol_link():
    os.symlink(r"C:/Users/philip.du/Desktop/test/10.md",r"F:/AI_Project/test1.md")

# This demo needs to be run from the repo folder.
# python demo/fake_gan/run.py
import random

import gradio as gr


# def fake_gan():
#     folder_path = r'F:/AI_Project/1'  # 指定文件夹路径
#     image_files = [f for f in os.listdir(folder_path) if f.endswith(('.jpg', '.jpeg', '.png'))]  # 加载所有图片文件
#     # images = [
#     #     (folder_path + f'/{file_name}')
#     #     for i, file_name in enumerate(image_files)
#     # ]
#     images = ['F:/AI_Project/1/1.png',
#               'F:/AI_Project/1/2.png',
#               'F:/AI_Project/1/3.png',
#               'F:/AI_Project/1/4.png',
#               'F:/AI_Project/1/5.png',
#               'F:/AI_Project/1/6.png']
#     print(images) 
#     return images



# with gr.Blocks() as demo:
#     with gr.Column(variant="panel"):
#         with gr.Row(variant="compact"):
#             text = gr.Textbox(
#                 label="Enter your prompt",
#                 show_label=False,
#                 max_lines=1,
#                 placeholder="Enter your prompt",
#             ).style(
#                 container=False,
#             )
#             btn = gr.Button("Generate image").style(full_width=False)

#         gallery = gr.Gallery(
#             label="Generated images", show_label=False, elem_id="gallery"
#         ).style(columns=[2], rows=[2], object_fit="contain", height="auto")

#     btn.click(fake_gan, None, gallery)

from flask import Flask, send_from_directory
import threading
app = Flask(__name__)
folder_path = "F:/AI_Project/1"

@app.route('/images/<path:filename>')
def serve_image(filename):
    return send_from_directory(folder_path, filename)

@app.route('/delete_image/<path:filename>', methods=['POST'])
def delete_image(filename):
    file_path = os.path.join(folder_path, filename)
    if os.path.exists(file_path):
        os.remove(file_path)
        return "Image deleted", 200
    else:
        return "Image not found", 404
def run_flask_server():
    app.run(port=8000)

# def fake_gan():
#     folder_path = r'F:/AI_Project/1'  # 指定文件夹路径
#     image_files = [f for f in os.listdir(folder_path) if f.endswith(('.jpg', '.jpeg', '.png'))]  # 加载所有图片文件
#     images = [
#         f'http://localhost:8000/images/{file_name}'
#         for i, file_name in enumerate(image_files)
#     ]
#     print(images)
#     return images

def fake_gan():
    folder_path = r'F:/AI_Project/1'  # 指定文件夹路径
    image_files = [f for f in os.listdir(folder_path) if f.endswith(('.jpg', '.jpeg', '.png'))]  # 加载所有图片文件
    image_tags = "".join([
        f'<div oncontextmenu="event.preventDefault();deleteImage(\'{file_name}\', event)" style="display:inline-block;position:relative;">'
        f'<a href="http://localhost:8000/images/{file_name}" target="_blank"><img src="http://localhost:8000/images/{file_name}" style="width:100px;height:auto;"></a>'
        f'</div>'
        for file_name in image_files
    ])
    delete_function = '''
    <script>
    async function deleteImage(filename, event) {
      if (confirm("Confirm delete?")) {
        const response = await fetch('http://localhost:8000/delete_image', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8'
          },
          body: 'filename=' + encodeURIComponent(filename)
        });
        if (response.ok) {
          event.target.parentElement.remove();
        } else {
          alert("Failed to delete image");
        }
      }
    }
    </script>
    '''
    return f'{image_tags}{delete_function}'
    
if __name__ == "__main__":
    # test_sysmbol_link()
    # create_junction(r"C:/Users/philip.du/Desktop/share",r"F:/AI_Project/test")
    flask_thread = threading.Thread(target=run_flask_server)
    flask_thread.start()
#     gallery = gr.Gallery(
#     label="Generated images", show_label=False, elem_id="gallery"
# ).style(columns=[4], rows=[4], object_fit="contain", height="auto")
    # iface = gr.Interface(fn=fake_gan, inputs=[], outputs=gallery)
    iface = gr.Interface(fn=fake_gan, inputs=[], outputs=gr.outputs.HTML())
    iface.launch(debug=True)
