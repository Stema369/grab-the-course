# grab-the-course
適用於逢甲選課系統<https://course.fcu.edu.tw/>的自動加選程式<br/>
程式功能:自動加選、衝堂加選、遇到異常(網路斷線、跳驗證碼...)自動重啟<br/>
加選頻率約1.4次/s<br/>
驗證碼辨識成功率約8成<br/>

### 不同方法比較
|方法|人工|滑鼠腳本|此專案(用selenium)|用requests|
|-|-|-|-|-|
|加選頻率(times/sec)|0.034|0.45|1.4|30|
|優點|使用無難度|啟動一次可以持續加選2~3個小時|可以穩定運行/加選頻率不算太高，公開後對學校系統影響不大(?)|可以最優先搶到課|
|缺點|手指痛/對於很熱門的課完全無用|用來加選的電腦，無法做其他事/不穩定(網路斷線、跳驗證碼、動到畫面)|啟動前要手動修改狀態|加選頻率太高，對學校系統影響大，較無法公開|

註:<br/>
人工加選頻率用一天3000次來算<br/>
滑鼠腳本加選頻率使用按鍵精靈來測試，加選間隔小於2.2秒容易跳驗證碼<br/>
用requests加選頻率30是依據[此文](https://www.dcard.tw/f/fcu/p/236947359)，不知道真的假的<br/>

### 加選狀態圖
1. 模式:加選新<br/>
![](https://upload.cc/i1/2022/09/04/o1yFbs.png)
2. 模式:舊換新<br/>
![](https://upload.cc/i1/2022/09/04/0sr8qL.png)

### Warning
***本人不保證程式完全無bug，若因程式bug造成使用者任何損失，皆須使用者自行承擔***

### 其他
本人已無使用此程式之需求，也不打算進行後續維護，如有需要歡迎fork到自己的github進行修改、優化、維護...，也歡迎公開分享自己修改後的版本

## How to use

### 事前準備
環境:<br/>
python 3.10+<br/>
1. 安裝相依模組<br/>
`pip3 install -r requirements.txt`<br/>
2. 安裝Tesseract OCR<br/>
<https://github.com/UB-Mannheim/tesseract/wiki><br/>
3. 修改453行左右的安裝路徑為剛才安裝tesseract.exe的路徑<br/>
```python
if __name__ == "__main__":
    os.environ["WDM_LOG_LEVEL"] = "0"
    pytesseract.pytesseract.tesseract_cmd = (
        r"你的安裝路徑..."
    )
```
4. 255行左右填寫你的學號、密碼<br/>
```python
def login():
    while 1:
        driver.get("https://course.fcu.edu.tw/")
        driver.find_element(By.XPATH, '//*[@id="ctl00_Login1_UserName"]').send_keys(
            "你的學號..."
        )
        driver.find_element(By.XPATH, '//*[@id="ctl00_Login1_Password"]').send_keys(
            "你的密碼..."
        )
```
5. 457行左右填寫你要加選的項目<br/>
	* 填寫範例:<br/>
	```python
	info = [
			{"模式": "加選新", "舊課": "0", "新課": "0123", "狀態": "新的選不到"},
			{"模式": "舊換新", "舊課": "0001", "新課": "0002", "狀態": "舊課程"},
	]
	```
	* 允許填寫的值:<br/>
	模式:加選新/舊換新<br/>
	舊課:0/(四碼選課代號)<br/>
	新課:(四碼選課代號)<br/>
	狀態:舊課程/舊的選不到/新課程/新的選不到<br/>
	***註:請勿填寫自己本系的必修課程***<br/>
	* 填寫模式、舊課、新課:<br/>
		* 加選的課程無衝堂，想要加選0001<br/>
		`模式:加選新 舊課:0 新課:0001`<br/>
		* 加選的課程有衝堂，衝堂的課為0001，想加選0002<br/>
		`模式:舊換新 舊課:0001 新課:0002`<br/>
		* 想加選0001與0002，但是兩堂課衝堂，且如果搶到0001之後想繼續搶0002<br/>
		`模式:舊換新 舊課:0001 新課:0002`<br/>
	* 填寫狀態範例:<br/>
		* 模式:加選新 舊課:0 新課:0001<br/>
		課表無0001，狀態=新的選不到<br/>
		課表有0001，狀態=新課程(或是把這整項刪了)<br/>
		* 模式:舊換新 舊課:0001 新課:0002<br/>
		課表沒有0001與0002，狀態=舊的選不到 or 新的選不到(無差別)<br/>
		課表只有0001，狀態=舊課程<br/>
		課表只有0002，狀態=新課程(或是把這整項刪了)<br/>

### 運行程式前須知
* 初次啟動前，需註解419行左右的兩行code，加選項目填一些不重要的課，然後運行程式並查看有無異常，確認無異常後，再去除註解並重新啟動<br/>
```python
#options.add_argument("headless")
#options.add_argument("disable-gpu")
```
* 程式重啟或是啟動前請核對加選項目與自己的課表，有需要時自行手動更改加選項目<br/>
* 程式開始執行後，除非console一直跳warning開頭的訊息，才需要停止並重啟，除此之外網路斷線之類的其他情況皆不須重啟<br/>
