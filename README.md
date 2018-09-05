# HalalStocksFilter

A class of Python functions used to automatically assess the halal-ness of a list of stocks.


## 1. The purpose of the HalalStocksFilter class

The purpose of the HalalStocksFilter class of functions is to automatically retrieve the data necessary to assess the halal-ness of a list of stocks (function: get_screen_data), and then produce a final output of stocks that pass and/or fail the standard Shariah Compliance test for tradeable stocks (apply_screen).


## 2. How it works

The way the get_screen_data function works is by scraping financial and text data from the Zacks website or the Wall Street Journal (WSJ) website (used as backup in case there are any difficulties with Zacks).

The apply_screen function runs quickly and applies some simple tests to a set of required financial data columns (the same columns retrieved from the get_screen_data function). 

The class is dependent on the following six packages:
- pandas
- datetime
- BeautifulSoup
- urllib
- json
- re

The information retrieved by get_screen_data are:
1)	Monthly Market Capitalization (from Zacks; no backup site available yet)
2)	Company Name (for user convenience; not essential)
3)	Company Description (from WSJ; with Zacks as backup)
4)	Latest Fiscal Year of data for items 4 and 5 below (from WSJ; with Zacks as backup)
5)	Cash and Short-Term Investments (from WSJ; with Zacks as backup)
6)	Total Accounts Receivable (from WSJ; with Zacks as backup)
7)	Latest Fiscal Year of data for items 7 and 8 below (from WSJ; with Zacks as backup)
8)	Non-Operating Interest Income (from WSJ; with Reuters as backup)
9)	Total Debt (from WSJ; with Zacks as backup))


The tests performed by apply_screen follow what are considered the standard Shariah compliance screens, as explained below (and as quoted from “Dow Jones Islamic Market Indices Methodology”): 

> Sector-Based Screens
Based on the Shariah Supervisory Board established parameters, the businesses listed below are inconsistent with Shariah law. The majority of Shariah scholars and boards hold that these industries and their financial instruments are inconsistent with Shariah precepts and hence are not suitable for Islamic investment purposes. Although no universal consensus exists among contemporary Shariah scholars on the prohibition of tobacco companies and the defense industry, most Shariah boards have advised against investment in companies involved in these activities. Income from the following impure sources cannot exceed 5% of revenue.
> - Alcohol
> - Tobacco
> - Pork-related products
> - Conventional financial services (banking, insurance, etc.)
> - Weapons and defense
> - Entertainment (hotels, casinos/gambling, cinema, pornography, music, etc.)

> Companies classified as Financial (8000) according to a unique proprietary classification
system are considered eligible if the company is incorporated as an Islamic Financial
Institution, such as:
> - Islamic Banks
> - Takaful Insurance Companies

> Companies classified as Real Estate (8600) according to a unique proprietary
classification system are considered eligible if the company’s operations and properties
are conducting business within Shariah principles.

> Accounting-Based Screens
After removing companies with unacceptable primary business activities, the remaining stocks are evaluated according to several financial ratio filters. The filters are based on criteria set up by the Shariah Supervisory Board to remove companies with unacceptable levels of debt or impure interest income. 
  
> All of the following must be less than 33%: 
> - Total debt divided by trailing 24-month average market capitalization  
> - The sum of a company’s cash and interest-bearing securities divided by trailing 24-month average market capitalization 
> - Accounts receivables divided by trailing 24-month average market capitalization 
 
*(Note that the apply_screen simply excludes ALL financial companies and real estate companies.)*
 

## 3. How it is used:

### get_screen_data(self, stocks, unique="No",  compression=None, verbose=0, User_Agent='')

This function only requires the stocks whose data we want to retrieve and returns a pandas dataframe and a file path to the location of that dataframe saved as a CSV. The returned dataframe will contain empty extra columns which are useful in case the same dataframe is used as input in the apply_screen

***stocks***: Either a list of stocks’ ticker symbols, or a dataframe which has a column that contains ticker symbols and whose column name is any of the following: 'Ticker', 'Symbol', 'Stock Symbol', 'Ticker Symbol', 'Stock Ticker'. If your column name is any of the following instead: 'Company', 'Company Name', 'Name', then that column is considered the full company name.

***unique***: If set to any value other than str(“No”) or str(“False”) or False, the output filename is appended with a distinct current date and time tag. Otherwise, it is appended with only the current date.

***compression***: Compression type in case the output file is to be compressed when written as a CSV.

***verbose***: Controls the amount of screen messages, and effectively ranges from 0 to 2 (anything greater than 2 is the same as 2).

***User_Agent***: Use this variable if user has a specific value for a website Request header, otherwise leave blank and a default value will be used when scraping.

-------------


### apply_screen(self, df_or_filename, mask=None, comp_desc=None, output=0, length=0, unique="No",  compression=None)

This function takes in a number of parameters and returns a dataframe of variable length and a file path to the location of that dataframe saved as a CSV. Depending on the input, it checks for if the following information is available, either in columns with the following names or columns that represent them: Company Description; Trailing 24-Month Average Market Capitalization (MkCap); Total Debt (TD); Cash & Short-Term Investments (Cash); Non-Operating Interest Income (Sec); Sum of Cash & Interest-Bearing Securities (Cash+Sec); Accounts Receivables (AR).
Note that the function needs to find at least either Cash+Sec or Cash and Sec separately.

***df_or_filename***: This input may either be a dataframe, or a filename leading to a dataframe which was saved in any of the standard file types available in the pandas package. If a dataframe is used, it will be checked to contain the required columns as listed above. If it is a dataframe produced by the get_screen_data function and no important column names are modified, then all the columns were detected. If this condition does not apply, for example because it is a custom user-made dataframe, then the mask parameter is needed for correct functioning.


***mask***: This parameter is either a list or a dict.

If mask is a list of numbers, its length should be 5 or 6, to represent the indices of the loaded dataframe's columns which represent the columns below, in their order below:

If the list is of length 5:
[Company Description ; Trailing 24-Month Average Market Capitalization (MkCap); Total Debt (TD); Sum of Cash & Interest-Bearing Securities (Cash+Sec); Accounts Receivables (AR)]

If the list is of length 6:
[Company Description; Trailing 24-Month Average Market Capitalization (MkCap); Total Debt (TD) ; Cash & Short Term Investments (Cash); Non-Operating Interest Income (Sec) ; Accounts Receivables (AR)]

If mask is a dict, make sure the keys are written as such, with values representing the indices of the loaded dataframe's columns of interest for the screening calculations:

{'CompDes':#, 'MkCap':#, 'TD':#, 'Cash_Sec':#, 'AR':#}

or as such:

{'CompDes':#, 'MkCap':#, 'TD':#, 'Cash':#,'Sec':#, 'AR':#}


***comp_desc***: This parameter contains key words to use against the company description for the Sector-based screening. The default value is None as the function contains a list of words, however the user can input their own list of words.


***output***: The default value is 0, but can take values [0, 1, >1]. A value of 0 gives only the positive (“Pass”) results. A value of 1 gives only the negative results (“Fail”) results. Any other value greater than 1 will give the full results (“Pass” and “Fail”).


***length***: The default value is 0, but can take values [0, 1, >1]. A value of 0 gives the minimal results: stock symbol and final result. A value of 1 gives a limited set of columns: 'Ticker', 'Description Screening', 'TD/MkCap < 33%','CashSec/MkCap < 33%', 'AR/MkCap Ratio < 33%','Fiscal Data Year (TD)','Fiscal Data Year (Sec)', 'Fiscal Data Year (Cash)','Fiscal Data Year (AR)','Final Result'. A value of greater than 1 gives the full set of columns.

***unique***: If set to any value other than str(“No”) or str(“False”) or False, the output filename is appended with a distinct current date and time tag. Otherwise, it is appended with only the current date.

***compression***: Compression type in case the output file is to be compressed when written as a CSV.


## 4. Example Usage with MagicFormula

A simple way of using the HalalStocksFilter alongside the MagicFormula function can be seen below:

>from MagicFormula import MagicFormula
>
>from HalalStocksFilter import HalalStocksFilter
>
>mf = MagicFormula.MagicFormula()
>
>hsf = HalalStocksFilter.HalalStocksFilter()
>
>stocks, filename = mf.get_stocks(email='example@example.com',password='example123',stocksNum=50)
>
>screen_df, filename2 = hsf.get_screen_data(stocks)
>
>final_df, filename3 = hsf.apply_screen(screen_df,length=2) # Using length=2 provides us with all the columns
