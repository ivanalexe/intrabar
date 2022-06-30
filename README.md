//@version=5
//Tradingview just introduced a new exciting ability to get intrabars candle information from a higher time frame chart!
//This indicator is based on Tradingview example but with enhanced capabilities to show the percentage of intrabars opposite bars with the ability to display the mid-day lower candle time frame close price drawn by a blue line.
indicator("Intrabar Polarity Divergences", max_labels_count =500,overlay = true)
time_frame= input.timeframe('5','Intrabar time frame')
show_absoulte_positive_nagative_values=  input.bool(true,'Show actual positive/nagative intrabar values')
show_mid_day_price = input.bool(true,'Show middle day intrabar close price')
array<float> directionsArray = request.security_lower_tf(syminfo.tickerid, time_frame, math.sign(close - open))
array<float> priceArray = request.security_lower_tf(syminfo.tickerid, time_frame, close)
varip int positive_bars=0
varip int nagative_bars=0
array_size=array.size(directionsArray)
varip float mid_bar_price = na
if (array_size>0)
    for i = 0 to array_size-1
    	if (array.get(directionsArray,i))>0
	        positive_bars+=1
	    if (array.get(directionsArray,i))<0
    	    nagative_bars+=1
    	if (math.round(array_size/2)==i) 
    	    mid_bar_price:=array.get(priceArray,i)
    	 
	    
rate_between_high_low =positive_bars>nagative_bars ? (positive_bars-nagative_bars)/positive_bars *100 :  (nagative_bars-positive_bars)/nagative_bars *100

bool is_different_direction=(math.sign(array.sum(directionsArray)) != math.sign(close - open) and math.sign(array.sum(directionsArray))!=0)
barcolor(is_different_direction ? color.new(color.orange,80) : na)
str_2= show_absoulte_positive_nagative_values ? '\n' +str.tostring(positive_bars)+"/"+str.tostring(nagative_bars) : ''
if (is_different_direction)
    label.new(bar_index,close ,xloc=xloc.bar_index,yloc= positive_bars>nagative_bars ? yloc.belowbar : yloc.abovebar, text=str.tostring(math.round_to_mintick(rate_between_high_low))+"%"+str_2, style=label.style_none,textcolor=positive_bars>nagative_bars ? color.green :color.red)

if (show_mid_day_price==false or is_different_direction==false)
    mid_bar_price:=na
plotbar(mid_bar_price,mid_bar_price,mid_bar_price,mid_bar_price)
if barstate.isnew
    positive_bars:=0
    nagative_bars:=0
    mid_bar_price:=na
