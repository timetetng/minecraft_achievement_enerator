# Minecraft 风格成就图片生成器

这是一个使用 Python 和 Pillow 库创建《我的世界》风格成就图片及完整成就看板的模块化工具。您可以轻松地将它集成到任何 Python 项目中，例如 Discord 机器人、服务器插件、Web 应用等，为您的用户提供美观、自动化的成就展示。
![成就预览](img\achi.png)
![成就看板预览](img\board.png)

## ✨ 功能特性

- **两种生成模式**：
    - 独立生成**单个成就**的横幅图片。
    - 根据用户数据，一键生成包含所有成就（已解锁和未解锁）的**完整成就看板**。
- **高度模块化**：
    - `AchievementGenerator`: 负责生成单个成就图片。
    - `AchievementBoardGenerator`: 负责生成总览看板，调用前者完成工作。
- **丰富的自定义选项**：
    - **稀有度系统**：内置8个稀有度等级（普通、稀有、史诗、传说、神话、奇迹、无瑕、未解锁），且完全可自定义。
    - **多彩标题**：支持为不同稀有度设置不同颜色，甚至支持**彩虹渐变**标题效果。
    - **像素级微调**：可自由配置标题垂直偏移、标题/描述的字间距。
- **智能文本处理**：
    - **自动换行**：精确的像素级换行算法，完美支持中英文混合长文本，杜绝溢出。
    - **自动过滤**：自动清除传入文本中可能存在的 Minecraft 颜色代码（如 `§c`）。
- **灵活的输出**：
    - 支持直接保存为**图片文件**。
    - 支持在内存中返回 **PIL Image 对象**，方便二次处理。
    - 支持返回 **Base64 编码字符串**，便于网络传输和网页嵌入。
- **数据驱动**：通过简单的字典/JSON结构来定义所有成就和用户数据，易于管理和扩展。

## ⚙️ 环境要求与安装

1.  **Python 环境**: 建议使用 Python 3.7 或更高版本。
2.  **Pillow 库**: 这是唯一的外部依赖。
    ```bash
    pip install Pillow
    ```
3.  **字体文件**:
    * 您需要一个像素风格的字体文件才能达到最佳效果。推荐使用 `Minecraftia` 或其他类似字体。
    * 请下载 `.ttf` 格式的字体文件，并将其与脚本放在一起（或提供其完整路径）。

## 🚀 使用方法

将项目中的 `achievement_system.py`（或您保存的文件名）复制到您的项目中，然后按需导入。

### 1. 生成单个成就

首先，实例化 `AchievementGenerator`。

```python
from achievement_system import AchievementGenerator

# 初始化生成器，传入字体路径
ach_gen = AchievementGenerator(font_path='minecraft_font.ttf')

# --- 示例 1：生成一个“稀有”成就并保存为文件 ---
ach_gen.create(
    title="钻石！",
    description="获得你的第一颗钻石。",
    icon_path="path/to/your/diamond_icon.png",
    theme='rare',
    output_format='file',
    output_path="output/diamond_achievement.png"
)

# --- 示例 2：生成一个“无瑕”成就，并获取 PIL Image 对象 ---
image_object = ach_gen.create(
    title="究极挑战者",
    description="完成所有进度。",
    icon_path="path/to/your/command_block_icon.png",
    theme='flawless',
    output_format='object'
)
# 现在你可以对 image_object 进行后续处理，例如：
# image_object.show()
# image_object.save("output/flawless_ach.png")
```

### 2. 生成成就看板 (核心功能)

这是为用户生成总览图的主要方式。

#### a. 准备数据

您需要准备两种数据结构：

-   **所有成就的定义 (字典)**：定义了游戏中所有可能存在的成就。
-   **用户的已解锁成就 (列表)**：一个包含用户已解锁成就ID的字符串列表。

```python
# 1. 所有成就的“数据库”
ALL_ACHIEVEMENTS_DATA = {
    "get_wood": {"title": "获得木头", "description": "采集第一个木头方块", "icon_path": "icons/wood.png", "rarity": "common"},
    "get_diamond": {"title": "钻石！", "description": "获得你的第一颗钻石", "icon_path": "icons/diamond.png", "rarity": "rare"},
    "kill_ender_dragon": {"title": "解放末地", "description": "击败末影龙", "icon_path": "icons/dragon_egg.png", "rarity": "miracle"}
    # ... 更多成就
}

# 2. 某个用户的已解锁成就ID列表
USER_UNLOCKED_IDS = [
    "get_wood",
    "get_diamond"
]
```

#### b. 生成看板

实例化 `AchievementBoardGenerator` 并调用 `create_board` 方法。

```python
from achievement_system import AchievementGenerator, AchievementBoardGenerator

# 1. 必须先初始化核心的 AchievementGenerator
ach_generator = AchievementGenerator(font_path='minecraft_font.ttf')

# 2. 再初始化看板生成器，将前者作为参数传入
board_generator = AchievementBoardGenerator(generator=ach_generator, font_path='minecraft_font.ttf')

# 3. 一键生成！
board_generator.create_board(
    user_name="Steve",
    all_achievements=ALL_ACHIEVEMENTS_DATA,
    unlocked_ids=USER_UNLOCKED_IDS,
    output_path="output/achievement_board_steve.png"
)
```

## 🎨 自定义与配置

所有的样式配置都集中在两个类的 `__init__` 方法中，您可以直接修改它们以符合您的需求。

### `AchievementGenerator` 自定义

```python
class AchievementGenerator:
    def __init__(self, font_path):
        # ...
        # 颜色配置 (可以添加或修改稀有度)
        self.TITLE_COLORS = { ... }
        self.FLAWLESS_GRADIENT_COLORS = [(255, 205, 26), (255, 46, 157)]

        # ...
        # 位置和间距微调
        self.title_y_adjust = -3       # 标题垂直偏移
        self.title_char_spacing = 1    # 标题字间距
        self.desc_char_spacing = 0     # 描述字间距
```

### `AchievementBoardGenerator` 自定义

```python
class AchievementBoardGenerator:
    def __init__(self, generator: AchievementGenerator, font_path: str):
        # ...
        # 看板字体
        self.board_font_title = ImageFont.truetype(font_path, 32)
        self.board_font_rarity = ImageFont.truetype(font_path, 24)

        # 看板布局
        self.board_width = 750         # 看板总宽度
        self.padding = 20              # 看板内外边距
        self.columns = 2               # 每行显示多少个成就
        self.grid_spacing_x = 20       # 成就之间的水平间距
        self.grid_spacing_y = 15       # 成就之间的垂直间距
        self.section_spacing = 30      # 不同稀有度区块之间的垂直间距
        self.bg_color = (50, 50, 50)   # 看板背景色
```

## 📁 文件结构建议

建议将相关文件按以下结构组织：

```
your_project/
├── your_main_script.py        # 你的主程序，在这里调用生成器
├── achievement_system.py      # 本工具的源代码
├── minecraft_font.ttf         # 字体文件
└── assets/
    └── icons/                 # 存放所有成就图标
        ├── wood.png
        ├── diamond.png
        └── ...
```

## 📜 许可证

本项目采用 [MIT License](https://opensource.org/licenses/MIT) 授权。您可以自由地使用、修改和分发。