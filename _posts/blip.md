### 理论：

1. Motivation

   1. `模型角度：`大多数方法要么采用基于编码器的模型，要么采用编码器-解码器模型。然而，基于编码器的模型不太容易直接转到文本生成任务（例如图像字幕），而编码器-解码器模型尚未成功用于图像文本检索任务。
   2. `数据角度: `SOTA的方法（如CLIP，ALBEF等）都在从web上收集到的图文对上进行预训练。尽管通过扩展数据集获得了性能提升，但文本的研究表明，对于视觉语言学习来说，有噪声的网络是次优的。

   为此，作者提出了BLIP：引导语言图像预训练，以实现统一的视觉理解和生成。BLIP是一个新的VLP框架，与现有方法相比，他可以实现更广泛的下游任务，从模型和数据角度有两个贡献：
2. Contribution

   1. `多模态编码器-解码器混合（MED）：`一种用于有效多任务预训练和灵活迁移学习的新模型架构。MED可以作为单模态编码器、基于图像的文本编码器或基于图像的文本解码器工作。该模型与三个视觉语言目标联合预训练：图像文本对比学习、图像文本匹配和图像条件语言建模。
   2. `字幕和过滤（CapFilt）：`一种新的数据集增强方法，用于从噪声图像-文本对中学习。作者将预先训练的MED分为两个模块: 一个**字幕器**，用于生成给定web图像的合成字幕，以及一个**过滤器**，用于从原始web文本和合成文本中删除嘈杂的字幕。
3. Model Architecture

### 复现

1. 数据下载需要在.py中寻找原url链接下载annotation得json文件
2. 安装ruamel_yaml库时需要使用conda 安装，或者在代码中修改为ruamel.yaml
3. 使用python<3.7版本的环境不行
4. 需要打开外网访问权限

   ```shell
   wget http://100.97.11.243:10086/proxy/enable_internet_proxy.sh
   bash enable_internet_proxy.sh
   source ~/.bashrc
   ```
5. 如果使用自己的数据集，预训练时则需要将json文件修改为

   ```Python
   {'image': path_of_image, 'caption': text_of_image}
   ```

- **如果是中文的数据集，则还需要改变文本的decode部分**
- 解决json文件路径与实际路径不对齐的方式：

  建立软连接将数据路径对齐
- batchsize 太大 8卡v100 75 内存溢出 设置为50
- 跳过下载预训练权重文件：

  ```shell
  mkdir -p /root/.cache/torch/hub/checkpoints/
  cp ./model_data/deit_base_patch16_224-b5f2ef4d.pth /root/.cache/torch/hub/checkpoints/
  ```
- 修改加载预训练权重路径

  ```python
  checkpoint = torch.hub.load_state_dict_from_url(
                  url="https://dl.fbaipublicfiles.com/deit/deit_base_patch16_224-b5f2ef4d.pth",
                  map_location="cpu", check_hash=True)
  ->
  checkpoint = torch.load('/data/ceph_11015/ssd/jintauhe/Blip/model_data/deit_base_patch16_224-b5f2ef4d.pth')

  ```

  以及bert模型的预训练权重

### 在中文表情包数据集上pretrain

难点：

1. 针对中文文本数据适应bert的中文预训练模型
2. 表情包的格式为.gif格式，如果使用

   ```python
   from PIL import Image 
   Image.open(image_path)
   ```
   会不会与RGB格式文件不一样，采取的是png格式的文件
3. config文件中需要变成中文的config文件，并且在后面加上item:`"add_cross_attention":true`
