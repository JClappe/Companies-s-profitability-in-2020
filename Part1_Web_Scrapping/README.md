# NASDAQ Company Profitability Impact

![001_NQ.jpg](attachment:001_NQ.jpg)

<font size = 6>Welcome to this new project!</font>

Today we will study the **impact of the current crysis on given companies**. The profit variation will be the endog variable and will be explained with foundamentals observations.\
Share price variation won't be added because predicting model has to work on unlisted companies. In other words, the work will only use public quaterly reports, which is mandatory for US listed company and must generalize well on every US companies with a minimal size/capital.

To do that, financials data will be get by web-scraping.\
Quaterly reports are only available on listed companies.\
**Companies included in NASDAQ** index are more than 3000 and **will firstly be studied**. More companies may be added if necessary.

Quaterly reports are scraped in totally free **Yahoo finance** instead of payant EDGAR API from SEC.

This part is only about web-scraping, which is dense enough.\
Steps in this part will be:
+ Get NASDAQ companies list with corresponding symbol
+ Get general information on each companies (sector,size,adress...)
+ Get quaterly report on each companies


I'm really happy to share this work with you...let's see what we can get! I wish you a good reading!


First let's import necessary librairies:


```python
%reset
```

    Once deleted, variables cannot be recovered. Proceed (y/[n])? y
    


```python
import pandas as pd
from bs4 import BeautifulSoup
import requests
import re
import numpy as np
import pandas as pd
import json
from time import time, sleep
from random import randint
from IPython.display import clear_output

from selenium import webdriver
from selenium.webdriver.support import expected_conditions as ec
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait

pd.set_option('display.max_columns',150)
```

# 1 - NASDAQ list

## 1.1 - Functions

[advfn website](https://www.advfn.com/nasdaq/nasdaq.asp?companies=A) is used to scrap NASDAQ list.

![002_NQ.JPG](attachment:002_NQ.JPG)

BeautifulSoup is the only necessary librairy. Every companies are listed in alphabetical order, so there are 26 pages to scrap, one for each letter.


```python
def NASDAQ_companies_name(pages):
    L_Name = []
    L_Symbol = []

    start_time = time()
    cpt = 0

    for page in pages:

        cpt+=1
        sleep(randint(3,6))


        url = 'https://www.advfn.com/nasdaq/nasdaq.asp?companies='+page
        response = requests.get(url)
        content = response.content
        print('Request {} >>> Status Code : {}'.format(cpt,response.status_code))

        parser = BeautifulSoup(content,'html.parser')
        selected = parser.find('table',class_='market tab1')

        class_type = 'ts'
        selectedts = selected.find_all('tr',class_=re.compile('ts'))
        nb_rows = len(selectedts)

        for row in np.arange(nb_rows):
            selected_row=selectedts[row]
            Informations = selected_row.find_all('a')
            if len(Informations)>1:
                Company_Name = Informations[0].text
                Company_Symbol = Informations[1].text
                L_Name.append(Company_Name)
                L_Symbol.append(Company_Symbol)

        eleapse_time = time() - start_time
        print('Request {} >>> Frequency: {} s/request'.format(cpt,round(eleapse_time/cpt,5)))
        clear_output(wait = True)

    df_NASDAQ = pd.DataFrame({'Name' : L_Name,'Symbol' : L_Symbol})
    return df_NASDAQ
```

## 1.2 - Extraction

Extraction is made with theses lines below, pages are called by letters in pages list. Scraping theses pages is very quick, it takes between 2 and 5 minutes.


```python
pages = ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','T','S','T','U','V','W','X','Y','Z','0']
df_NASDAQ_Name = NASDAQ_companies_name(pages)

df_NASDAQ_Name.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Symbol</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A. Schulman</td>
      <td>SHLM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A.c. Moore Arts Crafts</td>
      <td>ACMR</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A.d.a.m. Inc.</td>
      <td>ADAM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aaipharma</td>
      <td>AAII</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aaon</td>
      <td>AAON</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_NASDAQ_Name.to_csv('NASDAQ_Name.csv',index=None)
```


```python
df_NASDAQ_Name = pd.read_csv('NASDAQ_Name.csv')
df_NASDAQ_Name.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Symbol</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A. Schulman</td>
      <td>SHLM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A.c. Moore Arts Crafts</td>
      <td>ACMR</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A.d.a.m. Inc.</td>
      <td>ADAM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aaipharma</td>
      <td>AAII</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aaon</td>
      <td>AAON</td>
    </tr>
  </tbody>
</table>
</div>



Well done, the first dataset is made. It doesn't included meaningful informations for now but symbol will be used to get some.

# 2 - NASDAQ COMPANIES GENERAL INFOS

## 2.1 - Functions


[advfn website](https://ih.advfn.com/stock-market/NASDAQ/a-schulman-inc-delisted-SHLM/financials) is still used.

![003_NQ.JPG](attachment:003_NQ.JPG)

Get the needed outcome implies a little bit more complicated function. JSON is used and make it more clear.


```python
def clear_numeric(value):
    number = str()
    try: #If NaN or emply list is given
        for t in value:
            if (t.isnumeric()) | (t=='.'):
                number =number+ t

        try: #If number can't be converted to float
            value_to_return = float(number)
        except :
            value_to_return = np.NaN
    except:
        value_to_return = np.NaN
        
    return value_to_return

def brut_value (value):
    if str(value) =='':
        return np.NaN
    else:
        return value


def NASDAQ_companies_info(symbols):
    MarketCap = []
    Stocks_nb = []
    Dividends_5Y_P = []
    Holder_Institutionnal_P = []
    Holder_Insider_P = []
    DAV10 = []
    DAV50 = []
    alpha = []
    beta = []
    std = []
    R2 = []
    PER = []
    Company_Value = []
    Employees = []
    Industry = []
    Sector = []
    CIK = []
    Adress = []
    
    Symbols_ok = []
    L_missing_info = []
    
    cpt = 0
    start_time = time()
    
    for symbol in symbols:
        ###################### Tempo before requesting ######################
        cpt+=1
        sleep(randint(3,6))
        
        ###################### Requesting #####################
        url = 'https://ih.advfn.com/stock-market/NASDAQ/a-schulman-inc-delisted-'+symbol+'/financials'
        response = requests.get(url)
        content = response.content
        print('Status Code : {}'.format(response.status_code))

        parser = BeautifulSoup(content,'html.parser')
        
        try:
            #Get All Gathered informations in one time
            selected = parser.find('div',class_='container px-0').find('fundamentals-organism') #tpye : bs4.element.Tag
        except:
            print('{} info missing'.format(symbol))
            L_missing_info.append(symbol)
            continue
        selected_Dict = selected.attrs #All information in dict

        Keys = list(selected_Dict.keys())
    
        ####################################### INFORMATION SCRAPING ########################################
        # ============================================================================== Fundamentals informations
        Fundamentals = selected_Dict[Keys[0]] #Get one of the 3 important Key out of 5
        Dict_Fundamentals = json.loads(Fundamentals)

        Keys_Fundamentals = list(Dict_Fundamentals.keys())
        Share_information = Dict_Fundamentals[Keys_Fundamentals[0]]
        Dividend = Dict_Fundamentals[Keys_Fundamentals[1]]
        Holders = Dict_Fundamentals[Keys_Fundamentals[2]]
        TradingInfo = Dict_Fundamentals[Keys_Fundamentals[3]]

        # ============================================================================== KeyRatios
        KeyRatios = selected_Dict[Keys[1]] #Get one of the 3 important Key out of 5
        Dict_KeyRatios = json.loads(KeyRatios)
        Keys_KeyRatios = list(Dict_KeyRatios.keys())
        EvaluationMeasures = Dict_KeyRatios[Keys_KeyRatios[3]]

        # ============================================================================== Profile
        Profil = selected_Dict[Keys[2]] #Get the profil section
        Dict_Profil = json.loads(Profil) #Dict Profil
        Keys_Profil = list(Dict_Profil.keys())
        ProfilInfo = Dict_Profil[Keys_Profil[1]] #Profil Info in Profil dict
        Keys_ProfilInfo = list(ProfilInfo.keys())

        ContactInfo = Dict_Profil[Keys_Profil[2]] #Profil Info in Profil dict
        Keys_ContactInfo = list(ContactInfo.keys())

        #################### Fundamentals informations #########################
        # Share Info
        MarketCap.append(clear_numeric(list(Share_information.values())[0]))
        Stocks_nb.append(clear_numeric(list(Share_information.values())[1]))
        # Dividends
        Dividends_5Y_P.append(clear_numeric(list(Dividend.values())[1]))
        # Holding
        Holder_Institutionnal_P.append(clear_numeric(list(Holders.values())[3]))
        Holder_Insider_P = clear_numeric(list(Holders.values())[7])
        # Trading Info
        DAV10.append(clear_numeric(list(TradingInfo.values())[9])) #10 Day Average Volume 
        DAV50.append(clear_numeric(list(TradingInfo.values())[12])) #50 Day Average Volume
        alpha.append(clear_numeric(list(TradingInfo.values())[13])) #Alpha
        beta.append(clear_numeric(list(TradingInfo.values())[14])) #Beta
        std.append(clear_numeric(list(TradingInfo.values())[15])) #Standard Deviation
        R2.append(clear_numeric(list(TradingInfo.values())[16])) # R2


        #################### KEY RATIOS #########################
        PER.append(clear_numeric(list(EvaluationMeasures.values())[0]))
        Company_Value.append(clear_numeric(list(EvaluationMeasures.values())[1]))


        #################### PROFILE #########################
        Employees.append(ProfilInfo[Keys_ProfilInfo[4]])
        Industry.append(ProfilInfo[Keys_ProfilInfo[7]])
        Sector.append(ProfilInfo[Keys_ProfilInfo[8]])
        tmp_CIK = ProfilInfo[Keys_ProfilInfo[6]]
        CIK.append(tmp_CIK.split('>')[1].split('<')[0])

        Adress.append(ContactInfo[Keys_ContactInfo[0]])
        
        Symbols_ok.append(symbol)
        
        eleapse_time = time() - start_time
        print('Request {} >>> Frequency: {} s/request'.format(cpt,round(eleapse_time/cpt,5)))
        clear_output(wait = True)
        
    df_scraped = pd.DataFrame({'Name':Symbols_ok,
                               'MarketCap':MarketCap,
                               'Stocks_nb':Stocks_nb,
                               'Dividends_5Y_P':Dividends_5Y_P,
                               'Holder_Institutionnal_P':Holder_Institutionnal_P,
                               'Holder_Insider_P':Holder_Insider_P,
                               'DAV10':DAV10,
                               'DAV50':DAV50,
                               'alpha':alpha,
                               'beta':beta,
                               'std':std,
                               'R2':R2,
                               'PER':PER,
                               'Company_Value':Company_Value,
                               'Employees':Employees,
                               'Industry':Industry,
                               'Sector':Sector,
                               'CIK':CIK,
                               'Adress':Adress
                              })
    return df_scraped,L_missing_info
```

## 2.2 - Extraction

This function run with a list of companies symbol found with NASDAQ_companies_name.
Calling all symbols in one single list is a bad way to get needed data. About 3000 web pages are going to be scraped, doing this in one time is highly risky because of communication with server. Sometimes and for no logical reason, communication can be broke down.\
This type of errors might occurs and generate the exception below: 

![006_NQ.JPG](attachment:006_NQ.JPG)

SSLError appeared certainly because of TCP protocol error with the server. This phenomenon has nothing to do with our created function, it was caused by the internet connection or by the server itself.

Because of the high number of web page to scrap, the safest way to get all we want is to batch the symbol list. Regarding to the bad current web connection quality, NASDAQ_companies_info will be run with 30 batch of 100 companies.

One request take between 6 and 9 seconds (partially due the web connection), 3000 requests must be made, so 5 to 8 hours are needed on this step.


```python
threshold = 100
l_info_tmp = []
l_missing_tmp = []
for i in np.arange(df_NASDAQ_Name.shape[0]//threshold+1):
    NASDAQ_List_Symbol =  list(df_NASDAQ_Name['Symbol'])[i*threshold:(i+1)*threshold]
    df_NASDAQ_Info,L_missing_info = NASDAQ_companies_info(NASDAQ_List_Symbol)
    l_info_tmp.append(df_NASDAQ_Info)
    l_missing_tmp = l_missing_tmp + L_missing_info
```

    Status Code : 200
    Request 100 >>> Frequency: 6.0927 s/request
    


```python
df_Companies_info = pd.concat(l_info_tmp,axis='index')
df_Companies_info.reset_index(drop=True,inplace=True)
df_Companies_info.to_csv('Companies_infos.csv',index=None)
pd.Series(l_missing_tmp).to_csv('Companies_Missing_info.csv',index = None)

print('Companies Scraped : {}\n>>> Info obtained : {}\n>>> Info Missing  : {}'.format(df_NASDAQ_Name.shape[0],
                                                                                    df_Companies_info.shape[0],
                                                                                    len(l_missing_tmp)))
```

    Companies Scraped : 2996
    >>> Info obtained : 2410
    >>> Info Missing  : 586
    


```python
df_Companies_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>MarketCap</th>
      <th>Stocks_nb</th>
      <th>Dividends_5Y_P</th>
      <th>Holder_Institutionnal_P</th>
      <th>Holder_Insider_P</th>
      <th>DAV10</th>
      <th>DAV50</th>
      <th>alpha</th>
      <th>beta</th>
      <th>std</th>
      <th>R2</th>
      <th>PER</th>
      <th>Company_Value</th>
      <th>Employees</th>
      <th>Industry</th>
      <th>Sector</th>
      <th>CIK</th>
      <th>Adress</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SHLM</td>
      <td>1.298403e+09</td>
      <td>29509152.0</td>
      <td>1.02</td>
      <td>89.7</td>
      <td>0.0</td>
      <td>132571.0</td>
      <td>183891.0</td>
      <td>0.000928</td>
      <td>1.3699</td>
      <td>0.091445</td>
      <td>0.169341</td>
      <td>48.4</td>
      <td>2.164849e+09</td>
      <td>5300</td>
      <td>Chemicals</td>
      <td>Basic Materials</td>
      <td>0000087565</td>
      <td>3637 Ridgewood Road&lt;br/&gt;Fairlawn, OH 44333</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ACMR</td>
      <td>1.545660e+09</td>
      <td>18179954.0</td>
      <td>0.00</td>
      <td>38.9</td>
      <td>0.0</td>
      <td>577559.0</td>
      <td>571524.0</td>
      <td>0.106547</td>
      <td>0.8027</td>
      <td>0.277029</td>
      <td>0.022698</td>
      <td>92.4</td>
      <td>1.550265e+09</td>
      <td>361</td>
      <td>Semiconductors</td>
      <td>Technology</td>
      <td>0001680062</td>
      <td>42307 Osgood Road&lt;br/&gt;Suite I&lt;br/&gt;Fremont, CA ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAON</td>
      <td>2.999098e+09</td>
      <td>52031532.0</td>
      <td>11.60</td>
      <td>142.6</td>
      <td>0.0</td>
      <td>200438.0</td>
      <td>215379.0</td>
      <td>0.013401</td>
      <td>0.7492</td>
      <td>0.074896</td>
      <td>0.182908</td>
      <td>47.9</td>
      <td>3.007647e+09</td>
      <td>2290</td>
      <td>Construction</td>
      <td>Industrials</td>
      <td>0000824142</td>
      <td>2425 South Yukon Avenue&lt;br/&gt;Tulsa, OK 74107</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABAX</td>
      <td>1.898290e+09</td>
      <td>22870967.0</td>
      <td>0.00</td>
      <td>91.7</td>
      <td>0.0</td>
      <td>154925.0</td>
      <td>258341.0</td>
      <td>0.003428</td>
      <td>1.4398</td>
      <td>0.096504</td>
      <td>0.174023</td>
      <td>70.9</td>
      <td>1.731684e+09</td>
      <td>656</td>
      <td>Medical Diagnostics &amp; Research</td>
      <td>Healthcare</td>
      <td>0000881890</td>
      <td>3240 Whipple Road&lt;br/&gt;Union City, CA 94587</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AANB</td>
      <td>1.011362e+07</td>
      <td>3463569.0</td>
      <td>0.00</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>461.0</td>
      <td>0.0</td>
      <td>0.020162</td>
      <td>0.5091</td>
      <td>0.110400</td>
      <td>0.044413</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>102</td>
      <td>None</td>
      <td>None</td>
      <td>0000356809</td>
      <td>1130 Connecticut Ave. N.W.&lt;br/&gt;Washington, DC ...</td>
    </tr>
  </tbody>
</table>
</div>



# 3 - Annual Report NASDAQ

## 3.1 - Functions

Getting quaterly report clearly is the most complicated step. Two ways are possible:
+ SEC EDGAR API (payant)
+ Yahoo Finance (free)

[Yahoo Finance](https://fr.finance.yahoo.com/quote/AAON/financials?p=AAON) was chosen but some quaterly reports are missing.

![004_NQ.JPG](attachment:004_NQ.JPG)

**Financials elements and quaterly buttons must be clicked on to get the needed data.**\
Beautiful Soup can't be used in this case because of JavaScript routine. **The URL adress is the same between annual and quaterly report**, by defaut annual data is displayed and that's exactly what we don't want.\
Simulate quaterly button click is needed, **Selenium package can do this**. Scroll down also must be done to show the desired button in the web page and start JavaScript routine. That's why the command below is writen line 85:\
    browser.execute_script("window.scrollTo(0, 200)")


```python
############################ USED FUNCTION ANNUAL REPORT ##########################
def one_digit(value):
    for val in value:
        if val.isdigit():
            key = True
            break
        key = False
    try : 
        return key
    except:
        return np.NaN

import unicodedata
def split_num(value):
    splited = value.split(' ')
    splited_list = []
    for val in splited:
        splited_list.append(unicodedata.normalize('NFKD',val))

    return splited_list

def report_cleaning(report):
    report_tmp = []
    cpt = 0
    len_num_raw = []
    alpha_index = []
    for line in report:
        try:#======================== NAN
            if one_digit(line): 
                line_tmp = split_num(line)
                report_tmp.append(line_tmp)
                len_num_raw.append(len(line_tmp))
            else:
                txt = unicodedata.normalize('NFD', line).encode('ascii', 'ignore').decode('ascii','ignore')
                txt = txt.replace(' ','_')
                report_tmp.append(txt)
                alpha_index.append(cpt)
        except:
            report_tmp.append(np.NaN)
        cpt+=1
    return report_tmp,len_num_raw,alpha_index

def index_correction(RAW_list,RAW_alphas):
    
    #Determine index to pop
    to_pop = []
    for i in np.arange(0,len(RAW_alphas)):
        ##Pop index of numerical value which has no corresponding alpha value
        if i!=0:
            if (RAW_alphas[i]-RAW_alphas[i-1]!=2):
                for k in np.arange(1,RAW_alphas[i]-RAW_alphas[i-1]-1):
                    to_pop.append(RAW_alphas[i-1]+k)
                    
        #Pop index of alpha value which has no corresponding numerical value
        if i!=len(RAW_alphas)-1:       
            if RAW_alphas[i+1]==RAW_alphas[i]+1:
                to_pop.append(RAW_alphas[i])
        to_pop.sort()
        
        
    #Determine final list
    General_list = RAW_list.copy()
    biais = 0
    for i in to_pop:
        _=General_list.pop(i-biais)
        biais=biais+1
    return General_list


def Structure_Report(POPED_Report,len_num,symbol):
    POPED = POPED_Report.copy()
    # Arrange POPED report to dict
    dict_POPED = dict()
    for i in np.arange(0,len(POPED),2):
        dict_POPED[POPED[i]] = POPED[i+1]

    # Columns of the future final dataframe
    L_Structured = ['Total_revenue',
                    'Cost_of_revenue',
                    'Gross_profit',
                    'Selling_general_and_admin',
                    'Research_development',
                    'Total_operating_expenses',
                    'Operating_income_or_loss',
                    'Interest_expense',
                    'Total_other_income',
                    'Income_before_tax',
                    'Income_tax_expense',
                    'Net_income'
                   ]

    # Filling final dataframe
    dict_structured = dict()
    for categ in L_Structured:
        assigned = False
        for key,value in dict_POPED.items():
            if bool(re.search(categ,key)):
                assigned = True
                for q in np.arange(len_num[0]-1):
                    dict_structured[categ+'QM'+str(q)]=value[q+1]
        if assigned == False:
            for q in np.arange(len_num[0]-1):
                    dict_structured[categ+'QM'+str(q)]=0


    df_tmp_report = pd.DataFrame([dict_structured])
    df_Name = pd.DataFrame([symbol],columns=['Name'])
    return pd.concat([df_Name,df_tmp_report],axis='columns')

############################ END USED FUNCTION ANNUAL REPORT ##########################


def Get_Companies_Reports(list_symbol,time_sleep=1):
    time_to_wait = time_sleep
    browser = webdriver.Firefox()  #Initialising Firefox to Scrap
    L_All_Reports = []
    L_missing_report = []
    
    global_cpt = 0
    start_time = time()
    
    for symbol in list_symbol:
        try:
            ###################### Tempo before requesting ######################
            global_cpt+=1
            sleep(randint(3,6))

            url = 'https://uk.finance.yahoo.com/quote/'+symbol+'/financials?p='+symbol
            browser.get(url)


            # ================= Cookies Consent Box ===================
            try :
                #browser.find_element_by_class_name('con-wizard') # make sure the consent box is present
                box= WebDriverWait(browser, time_to_wait).until(ec.visibility_of_element_located((By.CLASS_NAME, "con-wizard")))
                browser.find_element_by_name('agree').click()
            except:
                pass

            
            # ===================================================== Find the 'trimestriel' button
            browser.execute_script("window.scrollTo(0, 200)") #Scroll to activate JS and find 'Quarterly' button
            buttons = browser.find_elements_by_css_selector('button')

            # There are two 'Quarterly' buttons, we want to click on the first
            button_to_push = []
            for i in np.arange(len(buttons)):
                if buttons[i].text == 'Quarterly':
                    button_to_push.append(i)

            buttons[button_to_push[0]].click() #click on the desired 'trimestriel' button (index 4)

            #Find desired reports 
            data= WebDriverWait(browser, time_to_wait).until(ec.visibility_of_element_located((By.XPATH, "//div[contains(@class,'D(tbrg)')]")))
            annual_report = data.text


            RAW_Report = annual_report.split('\n') #1st retake 
            CLEANED_Report,len_num,alphas = report_cleaning(RAW_Report) #Clean string report

            POPED_Report = index_correction(CLEANED_Report,alphas)

            # Verify the same len for all numeric rows
            same_value = (np.array(len_num)==len_num[0]).all()

            if same_value:
                L_All_Reports.append(Structure_Report(POPED_Report,len_num,symbol))
            else:
                print('{} not taken because of unequals numeric rows length'.format(symbol))

            eleapse_time = time() - start_time
            print('Request {} >>> Frequency: {} s/request'.format(global_cpt,round(eleapse_time/global_cpt,5)))
            clear_output(wait = True)
        except:
            print('{} not taken, missing report'.format(symbol))
            L_missing_report.append(symbol)
            continue
    df_All_Reports = pd.concat(L_All_Reports,axis='index')
    browser.close()
    return df_All_Reports,L_missing_report
```

The function globally make steps described below:
+ Open a Firefox browser and go to the Yahoo finance desired page                   **(line 114 to 130)**
+ Click ok in the cookies consent box, scroll down and click on the quaterly button **(line 131 to 150)**
+ Scrap the report data                                                             **(line 153 to 154)**
+ Structure raw data and put it in DataFrame                               **(line 1 to 108 and line 155 to line 172)**
+ Repeat for all companies
+ Quit Browser                                                                      **(line 136)**

Raw data unique shape implies a dense structuring process.

## 3.2 - Extraction

Get_Companies_Reports works the same way as the previous function. It takes Symbol list but return corresponding quaterly reports. The 3000 needed requests will be made by 30 batch of 100 symbols.


```python
threshold = 100
l_report_tmp = []
l_missing_report = []
for i in np.arange(df_NASDAQ_Name.shape[0]//threshold+1):
    NASDAQ_List_Symbol =  list(df_NASDAQ_Name['Symbol'])[i*threshold:(i+1)*threshold]
    NASDAQ_Report,l_missing = Get_Companies_Reports(NASDAQ_List_Symbol,time_sleep=15)
    l_report_tmp.append(NASDAQ_Report)
    l_missing_report = l_missing_report + l_missing
```


```python
df_Companies_report = pd.concat(l_report_tmp,axis='index')
df_Companies_report.reset_index(drop=True,inplace=True)
df_Companies_report.to_csv('Companies_report.csv',index=None)
pd.Series(l_missing_report).to_csv('Companies_Missing_report.csv',index = None)

print('Companies Scraped : {}\n>>> Info obtained : {}\n>>> Info Missing  : {}'.format(df_NASDAQ_Name.shape[0],
                                                                                    df_Companies_report.shape[0],
                                                                                    len(l_missing_report)))
```


```python
df_Companies_report.head()
```

**Congratulation to have come so far! But that's just the first step in a more complex work.**\
ETL process will be finished by creating a proper DataBase with SSIS and SQL Server Data Tool.

I hope you enjoyed reading this work.

This Kernel will always be a work in progress.\
If you want to discuss any other projects or just have a chat about data science topics, I'll be more than happy to connect with you on:
+ [LinkedIn](https://www.linkedin.com/in/jerome-clappe-3997b8149/)
+ [GitHub](https://github.com/JClappe)

See you and have a wonderful day!

![005_NQ.jpg](attachment:005_NQ.jpg)
