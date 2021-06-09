# ARWU-web-crawler
2021 Shanghai Ranking - [ **Global Ranking of Academic Subjects** ] ranking crawler

## Introduction
**Target** : ranking and score data with 54 subjects

- language : `python`
- IDE : `jupyter notebook`
- import package
    - BeautifulSoup
    - Selenium
    - requests
    - pandas
- output : `.csv` files 

- website : Shanghai Ranking of [Global Ranking of Academic Subjects](https://www.shanghairanking.com/rankings/gras/2021)


## Code

Step 0. Chrome driver
```python
from selenium.webdriver.support.ui import Select

option = Options()
option.add_argument("--disable-notifications")
chrome = webdriver.Chrome('./chromedriver', chrome_options=option)
```

Step 1. Get 54 subjects' name and their website ID.
Get content with class is `subject-title` as area title and class is `subj-link`'s href as subject link.
```python
url = "http://www.shanghairanking.com/rankings/gras/2021"
response = requests.get(url)
soup = BeautifulSoup(response.text, "lxml")

# area name: Natural Sciences / Engineering ...
area_list = []
areas = soup.find_all(class_="subject-title")
for area in areas:
    area_name = title.text.lstrip().rstrip()
    title_list.append(area_name)


# subject name: Mathematics / Physics
subject_dict = {}
subjects = soup.find_all(class_="subj-link")
for subject in subjects:
    # link format: /rankings/gras/2021/RSXXXX
    link = subject["href"].split("/")[-1]
    subject_dict[subject.text] = link
    print(link, subject.text)

# output: RS0101 Mathematics...
```

Step 2. test: Get number of the subject ranking
Extract all `<a>` tags and find the content text next to `•••` symbol.
```python
# find total page number
def get_final_page_number(sub_num):  # sub_num: RS0101

    # open website with seleium simulator and get content
    subject_url = "http://www.shanghairanking.com/rankings/gras/2021/" + sub_num
    chrome = webdriver.Chrome('./chromedriver', chrome_options=option)
    chrome.get(subject_url)
    soup = BeautifulSoup(chrome.page_source, 'lxml')

    tags_a = soup.find_all("a")
    flag, limited_page = False, 0

    # scan for all <a> tag
    for tag in tags_a:
        if(tag.text == "•••" and flag == False):
            flag = True
        elif(flag == True):
            try:
                limited_page = int(tag.text)  # get the content number
            except:
                print("ERROR: Error with convert page number to integer.")
            break

    print("limit:", limited_page)
    chrome.close()

    return limited_page
```

Step 3. test: Click ranking score item button
Find the button by using `find_element_by_class_name` method and `click()` it.


- score ranking button: `class = head-bg` (Q1 option button)
<img src="https://i.imgur.com/hBXIdh0.png" width=600>

```python
# there are two elements' class is 'head-bg', select the last one
btns = chrome.find_elements_by_class_name("head-bg", )
btns[-1].click()
```
- score ranking option item: `tag = <li>` with `Q1 / CNCI / IC / TOP / AWARD`
> For each score ranking option, find element with tag is 'li' and content is in btn_name_list

<img src="https://i.imgur.com/zNeZspJ.png" width=100>

```python
links = chrome.find_elements_by_tag_name("li")
btn_name_list = ["Q1", "CNCI", "IC", "TOP", "AWARD"]
for link in links:
    if(link.text == btn_name_list[index]):
        # select ranking score item button
        link.click() 
        # extract html content
        soup = BeautifulSoup(chrome.page_source, 'lxml')
        get_page_info(soup, btn_name_list[index])
        print(btn_name_list[index])
        time.sleep(2)
        break
```
Step 4. test: Click next page button
With multiple page buttons, select the last one with `class = ant-pagination-item-link`
> If you want to click the score ranking option in the next page afterwards, you should scroll the page to the top to continue.

<img src="https://i.imgur.com/B8UYtUw.png" height=50>

```python
# click the next page
btns_next_page = chrome.find_elements_by_class_name("ant-pagination-item-link", )
btns_next_page[-1].click()

# scroll page
js = "var action=document.documentElement.scrollTop=0"
chrome.execute_script(js)
```

Step 5. GO Crawler!!
