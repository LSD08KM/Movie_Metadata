## Bug修正


## 在cloud drive挂载的网络盘下，创建的nfo元数据没有被存储。

怀疑是不能对文件文本编辑。
改为在程序目录nfo文件夹下编辑好文件nfo_path_edit后，移动到指定路径nfo_path。

最近，上传小文件（封面、nfo）时出现 Reqwest error HTTP status client error (403 Forbidden) for url。如果先在115浏览器手动传过，就能成功上传。所以把nfo和图片都**保留一份本地缓存，再复制**到指定路径。之后手动上传再重试clouddrive队列。


core.py修改

```python
def print_files(path, leak_word, c_word, naming_rule, part, cn_sub, json_data, filepath, tag, actor_list, liuchu,
                uncensored, hack_word, _4k, fanart_path, poster_path, thumb_path):
    title, studio, year, outline, runtime, director, actor_photo, release, number, cover, trailer, website, series, label = get_info(
        json_data)
略
    try:
        if not os.path.exists(path):
            try:
                os.makedirs(path)
            except:
                print(f"[-]Fatal error! can not make folder '{path}'")
                os._exit(0)
        nfo_path_edit = os.path.join(".\cache", f"{number}{part}{leak_word}{c_word}{hack_word}.nfo")
        # 用于保留旧nfo评分信息
        old_nfo = None
略
        with open(nfo_path_edit, "wt", encoding='UTF-8') as code:
            print('<?xml version="1.0" encoding="UTF-8" ?>', file=code)
略
            print("  <website>" + website + "</website>", file=code)
            print("</movie>", file=code)
            code.close()
            print("[+]Wrote!            " + nfo_path_edit)
            shutil.copy(nfo_path_edit,nfo_path)
            print("[+]Moved!            " + nfo_path)
    except IOError as e:
略
```










# ！！！！还未设置！！！！ 新模式3增加判断字幕（srt或-c）,更新现有nfo标签和图片标签

模式3的无网络模式改成不更新图模式：原来是根据文件名（中文字幕后缀、流出后缀、破解后缀）和nfo更新图片水印。改为也更新nfo，但导致联网了，影响了参数-N --no-network-operation。除非不用get_data_from_json，直接写参数 json_data.get('naming_rule'), tag, json_data.get('actor_list')

create_data_and_move_with_custom_number时是用的core_main里的main mode 3，有下载图片

主.py修改

```python
def create_data_and_move(movie_path: str, zero_op: bool, no_net_op: bool, oCC):
略
            if no_net_op:
                core_main_no_net_op(movie_path, n_number, oCC)
略
                if no_net_op:
                    core_main_no_net_op(movie_path, n_number, oCC)
略            
```

core.py修改

```python
def core_main_no_net_op(movie_path, number, oCC, specified_source=None, specified_url=None):
略
	liuchu = False
    json_data = get_data_from_json(number, oCC, specified_source, specified_url)  # 定义番号
    # Return if blank dict returned (data not found)
    if not json_data:
        moveFailedFolder(movie_path)
        return
    tag = json_data.get('tag')    
略
        liuchu = True
    if 'hack'.upper() in str(movie_path).upper() or '破解' in movie_path:
        hack = True
        hack_word = "-hack"

略
    # 判断字幕文件，改写nfo
    # 给字幕文件加分段后缀
    if multi:
        number += part  # 这时number会被附加上CD1后缀    
    move_status = move_subtitles(movie_path, path, multi, number, part, leak_word, c_word, hack_word)
    if move_status:
        cn_sub = True
    # 最后输出.nfo元数据文件，以完成.nfo文件创建作为任务成功标志
    print_files(path, leak_word, c_word, json_data.get('naming_rule'), part, cn_sub, json_data, movie_path, tag,
                json_data.get('actor_list'), liuchu, uncensored, hack, hack_word
                , _4k, fanart_path, poster_path, thumb_path)
略

def core_main(movie_path, number_th, oCC, specified_source=None, specified_url=None):
略
    elif conf.main_mode() == 3:
        path = str(Path(movie_path).parent)
        if multi_part == 1:
            number += part  # 这时number会被附加上CD1后缀
        #判断subtitle文件
        move_status = move_subtitles(movie_path, path, multi_part, number, part, leak_word, c_word, hack_word)
        if move_status:
            cn_sub = True
略         
```

