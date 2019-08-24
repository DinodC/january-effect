
# Revisiting The January Effect

## Introduction
In this article, we review the infamous January effect which proposes that stocks' prices increase from December to January is the highest.
We illustrate the causes of the January effect, and present a simple trading strategy to profit from this calendar effect.
The project is shared on my online repository https://github.com/DinodC/january-effect.

We start off by importing packages


```python
import numpy as np
import pandas as pd
import pickle
```

## Pull Data

In this section, we collect S&P consittuents' historical data from a previous project https://quant-trading.blog/2019/06/24/backtesting-a-trading-strategy-part-2/.


```python
keys = ['sp500',
        'sp400',
        'sp600']
```

Initialize close


```python
close = {}
```

Pull data


```python
for i in keys:
    # Load OHLCV data
    with open(i + '_data.pickle', 'rb') as f:
        data = pickle.load(f)

    # Update close prices data
    close[i] = data.close
    
    f.close()
```

Inspect close prices of S&P 500 Index


```python
close['sp500'].head()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014-06-11</th>
      <td>40.1199</td>
      <td>40.2868</td>
      <td>125.0694</td>
      <td>86.0249</td>
      <td>45.0769</td>
      <td>66.4223</td>
      <td>22.92</td>
      <td>35.9633</td>
      <td>74.9719</td>
      <td>67.46</td>
      <td>...</td>
      <td>25.6691</td>
      <td>40.9150</td>
      <td>84.1321</td>
      <td>46.6073</td>
      <td>28.8084</td>
      <td>35.4555</td>
      <td>51.7899</td>
      <td>101.5068</td>
      <td>28.0164</td>
      <td>30.9798</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>39.7726</td>
      <td>38.2958</td>
      <td>123.2252</td>
      <td>84.5860</td>
      <td>44.6031</td>
      <td>65.9975</td>
      <td>23.03</td>
      <td>35.7925</td>
      <td>74.5018</td>
      <td>66.56</td>
      <td>...</td>
      <td>25.8547</td>
      <td>41.1019</td>
      <td>83.8928</td>
      <td>46.4230</td>
      <td>28.5372</td>
      <td>35.2780</td>
      <td>51.2950</td>
      <td>101.0367</td>
      <td>27.7535</td>
      <td>30.8448</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>39.8407</td>
      <td>38.4672</td>
      <td>123.7407</td>
      <td>83.6603</td>
      <td>45.0187</td>
      <td>66.2930</td>
      <td>22.95</td>
      <td>35.7745</td>
      <td>74.7820</td>
      <td>66.82</td>
      <td>...</td>
      <td>25.8884</td>
      <td>41.6091</td>
      <td>84.7098</td>
      <td>46.3744</td>
      <td>28.4920</td>
      <td>36.0532</td>
      <td>51.5945</td>
      <td>101.2861</td>
      <td>27.8192</td>
      <td>30.9702</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>39.7181</td>
      <td>39.1150</td>
      <td>124.0086</td>
      <td>84.5035</td>
      <td>44.8857</td>
      <td>66.0529</td>
      <td>23.27</td>
      <td>35.8734</td>
      <td>74.8634</td>
      <td>67.62</td>
      <td>...</td>
      <td>26.0740</td>
      <td>41.9028</td>
      <td>84.9326</td>
      <td>46.4424</td>
      <td>28.3791</td>
      <td>35.9318</td>
      <td>51.5099</td>
      <td>100.7105</td>
      <td>27.4060</td>
      <td>30.9798</td>
    </tr>
    <tr>
      <th>2014-06-17</th>
      <td>40.0927</td>
      <td>39.8867</td>
      <td>124.9411</td>
      <td>84.3935</td>
      <td>45.1351</td>
      <td>66.2561</td>
      <td>23.43</td>
      <td>35.8375</td>
      <td>74.5832</td>
      <td>67.54</td>
      <td>...</td>
      <td>26.0910</td>
      <td>42.2409</td>
      <td>84.5200</td>
      <td>46.2677</td>
      <td>28.8310</td>
      <td>36.0346</td>
      <td>51.7639</td>
      <td>100.4707</td>
      <td>27.9601</td>
      <td>31.6355</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>



For WordPress


```python
close['sp500'].head().iloc[:5, :5]
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014-06-11</th>
      <td>40.1199</td>
      <td>40.2868</td>
      <td>125.0694</td>
      <td>86.0249</td>
      <td>45.0769</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>39.7726</td>
      <td>38.2958</td>
      <td>123.2252</td>
      <td>84.5860</td>
      <td>44.6031</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>39.8407</td>
      <td>38.4672</td>
      <td>123.7407</td>
      <td>83.6603</td>
      <td>45.0187</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>39.7181</td>
      <td>39.1150</td>
      <td>124.0086</td>
      <td>84.5035</td>
      <td>44.8857</td>
    </tr>
    <tr>
      <th>2014-06-17</th>
      <td>40.0927</td>
      <td>39.8867</td>
      <td>124.9411</td>
      <td>84.3935</td>
      <td>45.1351</td>
    </tr>
  </tbody>
</table>
</div>




```python
close['sp500'].tail()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>67.95</td>
      <td>29.12</td>
      <td>154.61</td>
      <td>179.64</td>
      <td>76.75</td>
      <td>82.21</td>
      <td>267.06</td>
      <td>77.46</td>
      <td>177.97</td>
      <td>268.71</td>
      <td>...</td>
      <td>57.78</td>
      <td>106.90</td>
      <td>73.59</td>
      <td>54.40</td>
      <td>33.30</td>
      <td>77.40</td>
      <td>106.97</td>
      <td>117.41</td>
      <td>44.40</td>
      <td>108.12</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>68.35</td>
      <td>30.36</td>
      <td>154.61</td>
      <td>182.54</td>
      <td>77.06</td>
      <td>81.65</td>
      <td>268.80</td>
      <td>78.69</td>
      <td>179.56</td>
      <td>272.86</td>
      <td>...</td>
      <td>59.32</td>
      <td>105.60</td>
      <td>72.98</td>
      <td>55.38</td>
      <td>33.42</td>
      <td>78.88</td>
      <td>107.29</td>
      <td>118.54</td>
      <td>44.18</td>
      <td>108.50</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>69.16</td>
      <td>30.38</td>
      <td>154.90</td>
      <td>185.22</td>
      <td>77.07</td>
      <td>81.75</td>
      <td>269.19</td>
      <td>80.09</td>
      <td>180.40</td>
      <td>274.80</td>
      <td>...</td>
      <td>59.80</td>
      <td>106.01</td>
      <td>74.31</td>
      <td>55.63</td>
      <td>34.03</td>
      <td>79.15</td>
      <td>108.42</td>
      <td>120.31</td>
      <td>44.24</td>
      <td>108.89</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>69.52</td>
      <td>30.92</td>
      <td>155.35</td>
      <td>190.15</td>
      <td>77.43</td>
      <td>83.48</td>
      <td>267.87</td>
      <td>80.74</td>
      <td>182.92</td>
      <td>278.16</td>
      <td>...</td>
      <td>59.43</td>
      <td>107.49</td>
      <td>74.58</td>
      <td>55.94</td>
      <td>34.16</td>
      <td>79.56</td>
      <td>109.07</td>
      <td>120.73</td>
      <td>43.64</td>
      <td>110.06</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>70.29</td>
      <td>30.76</td>
      <td>153.52</td>
      <td>192.58</td>
      <td>76.95</td>
      <td>84.77</td>
      <td>272.43</td>
      <td>81.27</td>
      <td>184.44</td>
      <td>280.34</td>
      <td>...</td>
      <td>59.26</td>
      <td>110.88</td>
      <td>74.91</td>
      <td>57.10</td>
      <td>34.69</td>
      <td>80.38</td>
      <td>108.65</td>
      <td>121.71</td>
      <td>43.84</td>
      <td>110.22</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>



For WordPress


```python
close['sp500'].tail().iloc[:5, :5]
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>67.95</td>
      <td>29.12</td>
      <td>154.61</td>
      <td>179.64</td>
      <td>76.75</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>68.35</td>
      <td>30.36</td>
      <td>154.61</td>
      <td>182.54</td>
      <td>77.06</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>69.16</td>
      <td>30.38</td>
      <td>154.90</td>
      <td>185.22</td>
      <td>77.07</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>69.52</td>
      <td>30.92</td>
      <td>155.35</td>
      <td>190.15</td>
      <td>77.43</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>70.29</td>
      <td>30.76</td>
      <td>153.52</td>
      <td>192.58</td>
      <td>76.95</td>
    </tr>
  </tbody>
</table>
</div>




```python
close['sp500'].describe()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>...</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>52.010924</td>
      <td>41.207727</td>
      <td>145.418612</td>
      <td>135.094399</td>
      <td>66.653293</td>
      <td>84.553883</td>
      <td>162.741109</td>
      <td>49.113396</td>
      <td>118.473403</td>
      <td>142.243243</td>
      <td>...</td>
      <td>39.351443</td>
      <td>58.850843</td>
      <td>75.442714</td>
      <td>53.174206</td>
      <td>26.877544</td>
      <td>51.296575</td>
      <td>66.602743</td>
      <td>111.697803</td>
      <td>36.704361</td>
      <td>59.798250</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.866577</td>
      <td>6.366236</td>
      <td>24.128339</td>
      <td>38.127616</td>
      <td>17.721243</td>
      <td>9.483389</td>
      <td>116.577717</td>
      <td>12.653128</td>
      <td>31.189929</td>
      <td>70.413470</td>
      <td>...</td>
      <td>8.322974</td>
      <td>22.240068</td>
      <td>4.717698</td>
      <td>7.833835</td>
      <td>3.222176</td>
      <td>16.349757</td>
      <td>15.997628</td>
      <td>9.929764</td>
      <td>10.736900</td>
      <td>20.223068</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>82.743800</td>
      <td>42.066600</td>
      <td>65.718100</td>
      <td>22.220000</td>
      <td>33.935700</td>
      <td>68.852300</td>
      <td>60.880000</td>
      <td>...</td>
      <td>25.247700</td>
      <td>32.502600</td>
      <td>58.967500</td>
      <td>34.178400</td>
      <td>18.532600</td>
      <td>28.874500</td>
      <td>44.181800</td>
      <td>89.236100</td>
      <td>18.885300</td>
      <td>30.652000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>39.393500</td>
      <td>36.586125</td>
      <td>132.519725</td>
      <td>103.622650</td>
      <td>53.170800</td>
      <td>77.738125</td>
      <td>80.662500</td>
      <td>39.405975</td>
      <td>92.616475</td>
      <td>82.092500</td>
      <td>...</td>
      <td>31.485800</td>
      <td>42.102950</td>
      <td>72.534125</td>
      <td>48.731800</td>
      <td>24.215550</td>
      <td>35.033175</td>
      <td>53.060925</td>
      <td>103.343675</td>
      <td>26.826025</td>
      <td>44.749325</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>46.097600</td>
      <td>40.843300</td>
      <td>150.457700</td>
      <td>119.473800</td>
      <td>58.470800</td>
      <td>83.840300</td>
      <td>118.550000</td>
      <td>43.040600</td>
      <td>112.097050</td>
      <td>107.945000</td>
      <td>...</td>
      <td>39.055650</td>
      <td>52.368850</td>
      <td>75.817000</td>
      <td>54.521200</td>
      <td>26.573400</td>
      <td>48.085300</td>
      <td>61.564200</td>
      <td>112.761500</td>
      <td>38.198900</td>
      <td>51.333600</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>65.591850</td>
      <td>46.119900</td>
      <td>161.899875</td>
      <td>167.841200</td>
      <td>82.894300</td>
      <td>89.847025</td>
      <td>261.935000</td>
      <td>58.171250</td>
      <td>149.591750</td>
      <td>212.247500</td>
      <td>...</td>
      <td>45.444750</td>
      <td>68.789375</td>
      <td>78.491000</td>
      <td>59.650550</td>
      <td>29.650500</td>
      <td>67.341800</td>
      <td>80.202225</td>
      <td>119.131625</td>
      <td>46.431025</td>
      <td>80.809150</td>
    </tr>
    <tr>
      <th>max</th>
      <td>81.940000</td>
      <td>57.586600</td>
      <td>199.159900</td>
      <td>229.392000</td>
      <td>116.445400</td>
      <td>107.649700</td>
      <td>449.750000</td>
      <td>81.270000</td>
      <td>184.440000</td>
      <td>289.250000</td>
      <td>...</td>
      <td>59.800000</td>
      <td>139.263300</td>
      <td>86.137400</td>
      <td>67.795300</td>
      <td>35.000000</td>
      <td>83.549000</td>
      <td>109.070000</td>
      <td>130.912800</td>
      <td>57.139500</td>
      <td>110.220000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 505 columns</p>
</div>



For WordPress


```python
close['sp500'].describe().iloc[:5, :5]
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>52.010924</td>
      <td>41.207727</td>
      <td>145.418612</td>
      <td>135.094399</td>
      <td>66.653293</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.866577</td>
      <td>6.366236</td>
      <td>24.128339</td>
      <td>38.127616</td>
      <td>17.721243</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>82.743800</td>
      <td>42.066600</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>39.393500</td>
      <td>36.586125</td>
      <td>132.519725</td>
      <td>103.622650</td>
      <td>53.170800</td>
    </tr>
  </tbody>
</table>
</div>




```python
close['sp500'].shape
```




    (1258, 505)



## The January Effect
A calendar effect is an economic or stock market behavior which is related to the calendar such as the day of the week or the month of the year.
The most popular is the January effect which suggests that stock prices' increase from December to January is the highest.
The January effect was first observed by Sydney Wachtel in 1942, but seems to have lost its effect in recent years.

## Explanations Of The January Effect:
Possible explanations of the January effect include:
1. Tax-loss harvesting (or saving): 
Tax-loss harvesting allows investors to save taxes on realized gains using their unrealized losses.
In detail, investors sell the the losing stocks of their portfolio to generate losses.
Investors' taxes are reduced as the gains and losses are netted out.
2. Window dressing: 
Window dressing enables investors to improve the appearance of the portfolios which they manage.
In detail, investors buy (sell) winning (losing) stocks to enhance portfolio appearance.
Investors' portfolios has a better image as they now contain high-flying stocks.
3. Bonus: 
Bonus (end-of-year) allows investors to purchase stocks at the beginning of the year.
In detail, investors buying stocks in January will push the stock prices up.
Investors' bonus fuels price increase from December to January.

## A Trading Strategy To Profit From The January Effect
A possible trading strategy to profit from the January effect is the following:
1. Buy the losing (or off-loaded) stocks of December; and
2. Sell the winning stocks of December.

We backtest the strategy using the different investment universes:
1. S&P 500 Index composed of large capitalization stocks
2. S&P 400 Index comprised of medium capitalization stocks
3. S&P 600 Index composed of small capitalization stocks

Assume transaction cost (one-way)


```python
tc_one_way = 0.0005
```

Initialize the strategy's returns


```python
returns = {}
```

Backtesting the strategy


```python
for i in keys:
    # Create today series
    today = pd.Series(close[i].index)
    
    # Create months and years series
    months = pd.Series(close[i].index.month)
    years = pd.Series(close[i].index.year)
    
    # Create next day of the month and next day of the year
    next_day_month = months.shift(periods=-1)
    next_day_year = years.shift(periods=-1)
    
    # Last day of December
    mask_last_day_dec = (months==12) & (next_day_month==1)
    last_day_dec = today[mask_last_day_dec]
    
    # Last day of Jan
    mask_last_day_jan = (months==1) & (next_day_month==2)
    last_day_jan = today[mask_last_day_jan]
    
    # Ensure that last day of January is after last day of December
    assert (last_day_jan.values > last_day_dec.values).any(), 'Assertion violated'
    
    # End of year indices
    mask_eoy = (years!=next_day_year)
    eoy = today[mask_eoy]
    # Last item is not eoy
    eoy = eoy[:-1]
    
    # Check that eoy dates match last day of December dates
    assert (last_day_dec.values==eoy.values).any(), 'Assertion violated'
    
    # Calculate annual returns (from December of previous year to December of current year)
    annual_returns = close[i][mask_eoy.values].pct_change()
    
    # Retrieve last day of January close prices
    close_last_day_jan = close[i][mask_last_day_jan.values]
    
    # Retrieve last day of December close prices
    close_last_day_dec = close[i][mask_last_day_dec.values]
    # Modify the index of clost_last_day_dec 
    close_last_day_dec.index = close_last_day_jan.index
    
    # Calculate January returns (from December of previous year to January of current year)
    january_returns = (close_last_day_jan - close_last_day_dec) / close_last_day_dec
    
    for j in range(1, annual_returns.shape[0]-1):
        # Create a mask for stocks with returns != NaN
        mask_has_data = np.isfinite(annual_returns.iloc[j, :])
        has_data = list(mask_has_data[mask_has_data].index)

        # Sort stocks as per annual returns
        sort_tickers = annual_returns[has_data].iloc[j, :].sort_values().index

        # Set the number of stocks to long (short)
        top_n = round(len(has_data) / 10)

        # List of stocks to long and short
        longs = sort_tickers[:top_n]
        shorts = sort_tickers[-top_n:]

        # Calculate returns from the last day of December to the last day of January
        long_returns = (january_returns.iloc[j][longs]).mean()
        short_returns = (january_returns.iloc[j][shorts]).mean()
        portfolio_returns = 0.5 * (long_returns - short_returns) - 2 * tc_one_way

        # Update portfolio returns
        returns[i] = portfolio_returns
```

## Backtesting Results
In this section, we present the backtesting results of the trading strategy.


```python
pd.DataFrame(returns, 
            index=['returns'],
            columns=keys)
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
      <th>sp500</th>
      <th>sp400</th>
      <th>sp600</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>returns</th>
      <td>0.048986</td>
      <td>0.063328</td>
      <td>0.067413</td>
    </tr>
  </tbody>
</table>
</div>



Remarks:
1. The trading strategy generated positive returns under all investment universes.
2. The trading strategy produced the highest (lowest) returns using small (large) capitalization stocks.
3. Note that higher spreads are usually attached to small-cap stocks which could reduce the trading strategy's returns.

## Conclusion
In this article, we reviewed the well-known January effect, and proposed a trading strategy to profit from it.
The trading strategy buys (sells) the losing (winning) stocks in December, and awaits reversal in January.
The trading strategy generates the most (least) returns when using small (large) capitalization stocks.
