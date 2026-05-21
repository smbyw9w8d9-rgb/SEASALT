"""
智能错题本 · 三例题全预设版（含多错因选择与考前速记）
升级：
1. 彻底重构THOUGHT，去除重复拆分，贴合英语阅读真实做题心理；
2. 思考过程支持多选；
3. 正确答案+正确思路 ， 直接判定正确，不弹出偏差；
4. 多选思考 ， 匹配对应精准错因，去重展示，错因支持多选。
所有内部引号统一为中文全角“”，避免SyntaxError
"""

from collections import defaultdict

# ==================== 错因分类库（完全未改动） ====================
ERROR_CATEGORIES = {
    "1": {"name": "主旨大意错误", "sub": {"1": "以偏概全", "2": "偏离中心思想", "3": "内容/主旨混淆"}},
    "2": {"name": "细节理解错误", "sub": {"1": "定位错误", "2": "信息曲解", "3": "忽略否定词/限定词"}},
    "3": {"name": "推理判断错误", "sub": {"1": "过度推理", "2": "推理不足", "3": "脱离原文主观臆断"}},
    "4": {"name": "因果关系混淆", "sub": {"1": "倒置因果", "2": "忽略根本原因", "3": "强加因果"}},
    "5": {"name": "词汇理解错误", "sub": {"1": "词义猜测脱离上下文", "2": "熟词生义未识别", "3": "指代关系混淆"}},
    "6": {"name": "观点态度错误", "sub": {"1": "作者态度判断错误", "2": "人物观点张冠李戴"}}
}

# 考前速记提示（完全未改动）
CAUSE_MEMORY_TIPS = {
    ("1","1"): "主旨题以偏概全：别只看例子忘概括，考前提醒自己先找主题句。",
    ("1","2"): "主旨题偏离中心：别被次要信息带偏，注意首尾句和转折词。",
    ("1","3"): "内容/主旨混淆：内容常有例子和说明，但主旨需涵盖全文并上升到写作目的，注意最后一段作者呼吁读者怎么做。",
    ("2","1"): "细节定位错误：千万别急着选，没回原文精确定位，记得用题干关键词扫描，注意同义替换",
    ("2","2"): "信息曲解：千万别想当然.理解句子务必逐词对照原文。",
    ("2","3"): "忽略否定词/限定词：NOT/EXCEPT/only 等词总被跳过去，做题时圈出来。",
    ("3","1"): "过度推理：不要我觉得，要作者觉得。答案都从原文找依据，不脑补。",
    ("3","2"): "推理不足：别只看表面，没挖掘隐含逻辑，因果、目的要推一步。",
    ("3","3"): "主观臆断：脱离原文凭感觉，选择每个选项或者排除每个选项都要找到原文证据！！！！别偷懒！！！",
    ("4","1"): "倒置因果：不要把结果当原因，注意because/so/lead to的指向。",
    ("4","2"): "忽略根本原因：不要只看到直接事件，多问根本原因实什么。尝试去找主旨句，所有事例验证的那个句子。",
    ("4","3"): "强加因果：不要两件事无因果硬关联，检查是否有因果逻辑词。不要主观臆断",
    ("5","1"): "词义猜测脱离上下文：别根据构词法瞎猜，必须看前后句线索。",
    ("5","2"): "熟词生义未识别：有时候，常见词在文中是特殊含义，感觉奇怪就考虑熟词僻义的可能性，根据已知含义推测。考后去查，记住。",
    ("5","3"): "指代关系混淆：it/they/this 指什么没理清，画箭头标出。",
    ("6","1"): "作者态度判断错误：不要把人物观点当作者态度，区分文中不同声音。",
    ("6","2"): "人物观点张冠李戴：小心别谁说了什么记混，做题时标上人名缩写。"
}

# ==================== 例题数据（重写answer_thoughts + 同步更新thought_to_cause，其余未动） ====================

# --- 例题1：Tom迟到题（记叙文·原因题） ---
P1_TEXT = (
    "Tom was late for school because his alarm clock didn't ring. "
    "He ran to the bus stop, but the bus had already left. "
    "So he had to walk. When he finally arrived, the first class was over.\n"
    "Question: Why was Tom late for school?"
)
P1_ANSWERS = [
    "He missed the bus.",
    "His alarm clock didn't ring.",
    "He walked to school.",
    "The first class was over."
]
# 【重写】去除重复，按真实做题心理：关键词定位、浅层凭印象、深层逻辑颠倒
P1_ANSWER_THOUGHTS = {
    0: [
        "只看到表面事件，直接把中间发生的事当作迟到根本原因",
        "未梳理因果链，凭日常常识主观判断答案"
    ],
    1: [
        "定位because因果信号词，直接锁定原文根本原因",
        "梳理事件先后，区分直接事件与核心诱因"
    ],
    2: [
        "混淆因果逻辑，误将结果当作产生迟到的原因",
        "被后半段细节干扰，忽略题干核心提问方向"
    ],
    3: [
        "未理清事件时序，把迟到带来的结果当成原因",
        "扫读不细致，抓尾句信息忽略前文因果逻辑"
    ]
}

P1_WRONG_CAUSE = {
    0: [
        ("4", "2", "只关注到“bus left”这一中间结果，忽略了句首because引导的根本原因"),
        ("2", "1", "定位错误：看到了bus left就认为这是答案，没有继续找更早的原因"),
        ("3", "2", "推理不足：没有继续追问为什么巴士离开，未能追溯到闹钟故障")
    ],
    2: [
        ("4", "3", "强加因果：将“走路”误认为迟到原因，实际走路是结果不是原因"),
        ("2", "2", "信息曲解：误解了walk的作用，walk是错过了bus之后发生的事"),
        ("1", "2", "偏离中心：关注了次要细节“走路”，忽略了更重要的因果链")
    ],
    3: [
        ("2", "2", "信息曲解：误解了“first class was over”的含义，以为这导致迟到"),
        ("1", "1", "以偏概全：看到“first class was over”就以为和迟到直接相关"),
        ("3", "2", "推理不足：没有分析句子间的时序关系，第一节课结束是迟到的结果")
    ]
}

# 【同步重写】匹配新thought，1思维对应1–2个精准错因，无冗余
P1_THOUGHT_TO_CAUSE = {
    (0,0): [("4","2","只看中间事件，忽略原文根本原因")],
    (0,1): [("3","3","脱离原文，凭常识主观臆断因果")],
    (2,0): [("4","3","强加因果，混淆结果与原因")],
    (2,1): [("1","2","偏离核心，被次要细节带偏")],
    (3,0): [("2","2","信息曲解，颠倒事件因果时序")],
    (3,1): [("3","2","推理不足，未挖掘深层逻辑关系")]
}

P1_RIGHT_THOUGHT = {
    1: [0,1]
}

P1_THOUGHT_CAUSE = {
    0: ("4", "2", "思维停留在中间事件，未追溯到根本原因"),
    1: None,
    2: None
}
P1_REF_STEPS = [
    "1. 定位问题焦点：迟到的原因",
    "2. 扫读全文寻找因果信号词 (because, so)",
    "3. 发现首句 'because his alarm clock didn't ring'",
    "4. 识别 'missed the bus' 是中间结果，非根本原因",
    "5. 理清因果链：闹钟没响 → 错过公交 → 走路 → 迟到",
    "6. 得出结论：根本原因是闹钟没响"
]
P1_CORRECT_IDX = 1

# --- 例题2：沙漠植物说明文（主旨题） ---
P2_TEXT = (
    "Plants in the desert have developed amazing ways to survive. "
    "Cacti turn their leaves into spines to reduce water loss. "
    "The roots of camel thorn trees can grow as deep as 30 meters to find underground water. "
    "Some plants drop their leaves during dry seasons and remain dormant until rain returns. "
    "These adaptations allow them to live in extreme environments.\n"
    "Question: What is the main idea of the passage?"
)
P2_ANSWERS = [
    "Desert plants have special ways to survive.",
    "Cacti have spines to reduce water loss.",
    "Some plants drop their leaves in dry seasons.",
    "Desert environments are very harsh."
]
# 【重写】主旨题真实思维：抓首尾主旨句、被例子带偏、主观臆断主题
P2_ANSWER_THOUGHTS = {
    0: [
        "快速识别首尾主题句，概括全文核心",
        "区分主旨与举例细节，不被局部信息干扰"
    ],
    1: [
        "被文中典型例子吸引，误将细节当作全文主旨",
        "不会区分例证与中心，缺乏整体概括意识"
    ],
    2: [
        "凭印象抓取局部信息，以单个细节概括全文",
        "未定位主题句，碎片化理解文章内容"
    ],
    3: [
        "误判文章描述主体，偏离植物适应的核心话题",
        "过度放大次要信息，主观臆断写作目的"
    ]
}

P2_WRONG_CAUSE = {
    1: [
        ("1", "1", "以偏概全：只关注仙人掌这一个例子，忽略了其他植物"),
        ("2", "2", "信息曲解：cactus spines只是细节，把它当成了文章主旨"),
        ("1", "2", "偏离中心思想：被例子吸引，忘了文章第一句才是主题")
    ],
    2: [
        ("1", "1", "以偏概全：掉叶子只是其中一种适应方式，不能代表主旨"),
        ("2", "1", "定位错误：凭印象选了记住的细节，没有回到主题句"),
        ("3", "3", "主观臆断：觉得这个细节很突出，就以为是文章中心")
    ],
    3: [
        ("1", "2", "偏离中心思想：文章讲植物如何适应，而非环境本身有多恶劣"),
        ("3", "3", "脱离原文主观臆断：将文中的“极端环境”过度放大成主旨"),
        ("2", "2", "信息曲解：把最后一句的“extreme environments”误作主题")
    ]
}

# 【同步重写】匹配新thought
P2_THOUGHT_TO_CAUSE = {
    (1,0): [("1","1","以偏概全，把例子当作主旨")],
    (1,1): [("1","2","偏离中心，缺乏整体概括能力")],
    (2,0): [("1","1","局部代替整体，以偏概全")],
    (2,1): [("2","1","定位错误，未查找主题句")],
    (3,0): [("1","2","偏离文章核心描述主体")],
    (3,1): [("3","3","脱离原文，主观臆断主题")]
}

P2_RIGHT_THOUGHT = {
    0: [0,1]
}

P2_THOUGHT_CAUSE = {
    0: None,
    1: ("1", "1", "以偏概全，把例子当成主旨"),
    2: ("3", "2", "推理不足，未识别主题句")
}
P2_REF_STEPS = [
    "1. 阅读首尾句，寻找主题句",
    "2. 识别文章结构：主题句 + 例子支撑",
    "3. 确认第一句为全文中心",
    "4. 判断选项A概括了全部例子，其余为细节",
    "5. 选择A"
]
P2_CORRECT_IDX = 0

# --- 例题3：社交媒体议论文（细节NOT题） ---
P3_TEXT = (
    "Social media has changed the way teenagers communicate. "
    "While it helps them stay connected with friends, it also brings problems. "
    "Studies show that spending too much time on social media can lead to anxiety and sleep loss. "
    "However, not all effects are negative. Many teens use platforms to share creative work and find support communities. "
    "Experts suggest that the key is balanced use: limiting screen time and focusing on real-life interactions.\n"
    "Question: According to the passage, which of the following is NOT mentioned as a positive effect of social media?"
)
P3_ANSWERS = [
    "Staying connected with friends.",
    "Sharing creative work.",
    "Finding support communities.",
    "Improving academic performance."
]
# 【重写】细节否定题真实思维：忽略NOT、审题不全、精准反向筛选
P3_ANSWER_THOUGHTS = {
    0: [
        "忽略题干NOT限定词，只匹配原文正向信息",
        "审题不完整，未关注题干特殊提问要求"
    ],
    1: [
        "识别选项为原文正面信息，忽略题目选未提及项",
        "未做反向排除，只做正向细节匹配"
    ],
    2: [
        "精准定位原文，但忘记反向筛选NOT要求",
        "细节核对正确，审题逻辑出现偏差"
    ],
    3: [
        "标记NOT关键词，逐项反向排除原文信息",
        "严谨审题，精准核对选项与题干要求"
    ]
}

P3_WRONG_CAUSE = {
    0: [
        ("2", "3", "忽略了问题中的“NOT”，将提到的积极影响误选为答案"),
        ("2", "1", "定位错误：看到 stay connected 与原文相符，却没核对题干否定要求"),
        ("3", "2", "推理不足：没有意识到问题要求选“未提及”的，盲目根据熟悉度选择")
    ],
    1: [
        ("2", "3", "忽略了问题中的“NOT”，将提到的积极影响误选为答案"),
        ("2", "1", "定位错误：只确认选项与原文一致，忘记检查问题限定条件"),
        ("1", "1", "以偏概全：只看到 sharing creative work 是正面，忽略问题问的是NOT")
    ],
    2: [
        ("2", "3", "忽略了问题中的“NOT”，将提到的积极影响误选为答案"),
        ("2", "2", "信息曲解：将“support communities”当成答案，实际是文中提到的positive"),
        ("3", "2", "推理不足：未把题干NOT与选项做反向筛选")
    ]
}

# 【同步重写】匹配新thought
P3_THOUGHT_TO_CAUSE = {
    (0,0): [("2","3","忽略NOT否定限定，审题失误")],
    (0,1): [("2","3","审题不全，忽略题干特殊要求")],
    (1,0): [("2","3","忽略否定词，未做反向筛选")],
    (1,1): [("3","2","推理不足，只做正向匹配")],
    (2,0): [("2","3","审题偏差，忘记反向筛选")],
    (2,1): [("3","2","细节核对正确，逻辑推理不足")]
}

P3_RIGHT_THOUGHT = {
    3: [0,1]
}

P3_THOUGHT_CAUSE = {
    0: ("2", "3", "忽略题干否定词，审题失误"),
    1: ("2", "3", "忽略NOT，审题不严谨"),
    2: ("2", "3", "审题偏差，忽略限定条件"),
    3: None
}
P3_REF_STEPS = [
    "1. 审题：问题要求选出文中 NOT mentioned 的 positive effect",
    "2. 回原文定位 positive effects: stay connected, share creative work, find support communities",
    "3. 逐一比对选项，发现D 'improving academic performance' 未提及",
    "4. 确认A、B、C均为原文细节",
    "5. 选择D"
]
P3_CORRECT_IDX = 3

PROBLEMS = [
    {"name": "Tom迟到题（记叙文·原因题）", "text": P1_TEXT, "answers": P1_ANSWERS,
     "answer_thoughts": P1_ANSWER_THOUGHTS, "wrong_cause": P1_WRONG_CAUSE,
     "thought_to_cause": P1_THOUGHT_TO_CAUSE, "right_thought": P1_RIGHT_THOUGHT,
     "thought_cause": P1_THOUGHT_CAUSE, "ref_steps": P1_REF_STEPS, "correct": P1_CORRECT_IDX},
    {"name": "沙漠植物题（说明文·主旨题·英文）", "text": P2_TEXT, "answers": P2_ANSWERS,
     "answer_thoughts": P2_ANSWER_THOUGHTS, "wrong_cause": P2_WRONG_CAUSE,
     "thought_to_cause": P2_THOUGHT_TO_CAUSE, "right_thought": P2_RIGHT_THOUGHT,
     "thought_cause": P2_THOUGHT_CAUSE, "ref_steps": P2_REF_STEPS, "correct": P2_CORRECT_IDX},
    {"name": "社交媒体题（议论文·细节题·英文）", "text": P3_TEXT, "answers": P3_ANSWERS,
     "answer_thoughts": P3_ANSWER_THOUGHTS, "wrong_cause": P3_WRONG_CAUSE,
     "thought_to_cause": P3_THOUGHT_TO_CAUSE, "right_thought": P3_RIGHT_THOUGHT,
     "thought_cause": P3_THOUGHT_CAUSE, "ref_steps": P3_REF_STEPS, "correct": P3_CORRECT_IDX}
]

# ==================== 工具函数（完全未改动） ====================
def show_statistics(stats):
    if not stats:
        print("  暂无记录。")
        return
    print("\n📊 当前错因统计：")
    total = sum(stats.values())
    for error_type, count in sorted(stats.items(), key=lambda x: x[1], reverse=True):
        bar = "█" * count
        print(f"  {error_type}: {count} {bar}")
    print(f"  总计错误次数：{total}")

def show_quick_memory(stats):
    if not stats:
        print("暂无记录，无速记提示。")
        return
    print("\n🧠 考前速记（你的薄弱点及提醒）：")
    for error_type, count in stats.items():
        parts = error_type.split(" - ")
        if len(parts) != 2:
            continue
        cat_name, sub_name = parts[0], parts[1]
        cause_key = None
        for key, cat in ERROR_CATEGORIES.items():
            if cat["name"] == cat_name:
                for subkey, sub in cat["sub"].items():
                    if sub == sub_name:
                        cause_key = (key, subkey)
                        break
                break
        if cause_key and cause_key in CAUSE_MEMORY_TIPS:
            tip = CAUSE_MEMORY_TIPS[cause_key]
            print(f"  📌 {error_type}（出现{count}次）：{tip}")

def choose_from_list(prompt, options, allow_multi=False):
    print(prompt)
    for i, opt in enumerate(options, 1):
        print(f"  {i}. {opt}")
    if allow_multi:
        print("💡 可多选，输入多个编号用英文逗号分隔，例如：1,3")
    while True:
        choice = input("请输入选项编号：").strip()
        if allow_multi:
            parts = choice.split(",")
            idxs = []
            valid = True
            for p in parts:
                if not p.isdigit():
                    valid = False
                    break
                num = int(p)
                if not (1 <= num <= len(options)):
                    valid = False
                    break
                idxs.append(num-1)
            if valid:
                return idxs
            print("❌ 输入无效，请重新输入（多个用英文逗号分隔）")
        else:
            if choice.isdigit():
                idx = int(choice)
                if 1 <= idx <= len(options):
                    return [idx - 1]
            print("❌ 无效输入，请重新输入数字。")

def record_error(stats, cause_tuple_list):
    for cat_key, sub_key in cause_tuple_list:
        error_label = f"{ERROR_CATEGORIES[cat_key]['name']} - {ERROR_CATEGORIES[cat_key]['sub'][sub_key]}"
        stats[error_label] += 1
        print(f"✅ 已记录：{error_label}")

def manual_choose_cause():
    print("\n🔍 手动选择错因（一级大类 → 二级子类）")
    cat_keys = list(ERROR_CATEGORIES.keys())
    cat_names = [ERROR_CATEGORIES[k]["name"] for k in cat_keys]
    idx1 = choose_from_list("请选择一级错误类型：", cat_names)[0]
    cat_key = cat_keys[idx1]
    sub_keys = list(ERROR_CATEGORIES[cat_key]["sub"].keys())
    sub_names = list(ERROR_CATEGORIES[cat_key]["sub"].values())
    idx2 = choose_from_list("请选择二级具体错因：", sub_names)[0]
    sub_key = sub_keys[idx2]
    return (cat_key, sub_key)

def analyze_problem(prob, stats):
    print("\n" + "=" * 60)
    print(f"📖 {prob['name']}")
    print("=" * 60)
    print(prob['text'])
    print("=" * 60)

    # 1.选择答案（单选）
    ans_idx = choose_from_list("✏️ 请选择你的答案：", prob['answers'])[0]
    student_answer = prob['answers'][ans_idx]

    # 2.选择思考过程【支持多选】
    print("\n💭 第一步：选择你做题时真实的思考过程（可多选）")
    thought_options = prob["answer_thoughts"][ans_idx]
    thought_idxs = choose_from_list("", thought_options, allow_multi=True)

    # 正确答案逻辑：判断选中的思路是否全部为正确思路
    if ans_idx == prob['correct']:
        right_thought_set = set(prob["right_thought"].get(ans_idx, []))
        all_right = all(t in right_thought_set for t in thought_idxs)
        if all_right:
            print(f"\n✅ 你的答案【{student_answer}】正确，思路完全正确！")
            really = input("你是真的完全理解了吗？(y=是，n=其实还有点蒙)：").strip().lower()
            if really != 'y':
                print("请手动选择薄弱错因用于统计：")
                manual_cause = manual_choose_cause()
                record_error(stats, [manual_cause])
        else:
            # 有部分思路存在偏差
            print(f"\n✅ 你的答案【{student_answer}】正确，但部分思考存在偏差。")
            for t in thought_idxs:
                cause = prob['thought_cause'].get(t)
                if cause is not None:
                    cat_key, sub_key, desc = cause
                    print(f"⚠️ 偏差：{desc}")
            auto_idx = choose_from_list("是否记录这些偏差？", ["记录", "不记录", "手动自选错因"])
            if auto_idx[0] == 0:
                causes = []
                for t in thought_idxs:
                    c = prob['thought_cause'].get(t)
                    if c:
                        causes.append((c[0], c[1]))
                record_error(stats, causes)
            elif auto_idx[0] == 2:
                manual_cause = manual_choose_cause()
                record_error(stats, [manual_cause])
        print("\n📝 【参考思维路径】")
        for step in prob['ref_steps']:
            print(f"  {step}")

    else:
        # 错误答案：多选思考 → 匹配所有对应错因，去重
        print(f"\n❌ 你的答案【{student_answer}】不正确。")
        matched_causes = []
        seen = set()
        for t in thought_idxs:
            key = (ans_idx, t)
            clist = prob["thought_to_cause"].get(key, [])
            for c in clist:
                ck, sk, desc = c
                if (ck, sk) not in seen:
                    seen.add((ck, sk))
                    matched_causes.append(c)
        if matched_causes:
            print("\n🤖 根据你刚才的思考，匹配到以下错因（可多选）：")
            desc_list = [desc for _,_,desc in matched_causes]
            chosen_idxs = choose_from_list("", desc_list, allow_multi=True)
            chosen_causes = [matched_causes[i][:2] for i in chosen_idxs]
            confirm = input(f"\n确认记录这些原因？(y/n)：").strip().lower()
            if confirm == 'y':
                record_error(stats, chosen_causes)
            else:
                print("⏭️ 跳过本题记录。")
        else:
            print("无匹配错因，请手动选择。")
            chosen_causes = [manual_choose_cause()]
            record_error(stats, chosen_causes)

        print("\n📝 【参考思维路径】")
        for step in prob['ref_steps']:
            print(f"  {step}")

    print("\n💡 请花10秒反思自己的思维与标准路径的差异。")

def main():
    stats = defaultdict(int)
    print("=" * 60)
    print("📚 英语阅读错因自诊系统（重写真实做题思维+多选+精准匹配）")
    print("=" * 60)

    while True:
        print("\n📌 请选择题号：")
        for i, prob in enumerate(PROBLEMS, 1):
            print(f"  {i}. {prob['name']}")
        print("  q. 退出")
        choice = input("输入数字或q：").strip()
        if choice.lower() == 'q':
            break
        if choice.isdigit():
            idx = int(choice)
            if 1 <= idx <= len(PROBLEMS):
                analyze_problem(PROBLEMS[idx-1], stats)
            else:
                print("❌ 无效题号。")
                continue
        else:
            print("❌ 无效输入。")
            continue

        while True:
            print("\n➡️ 下一步：")
            print("  c. 继续选新题")
            print("  s. 查看当前统计")
            print("  m. 考前速记")
            action = input("请输入 (c/s/m)：").strip().lower()
            if action == 'c':
                break
            elif action == 's':
                show_statistics(stats)
            elif action == 'm':
                show_quick_memory(stats)
            else:
                print("❌ 无效输入。")

    print("\n🏁 最终统计：")
    show_statistics(stats)
    print("\n🧠 考前速记（你的薄弱点）：")
    show_quick_memory(stats)
    print("\n🎉 练习结束！")

if __name__ == "__main__":
    main()
