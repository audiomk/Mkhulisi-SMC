//@version=3
study("3 Moving Averages by RayGizmo", shorttitle="3MA - RG", overlay=true)
price=input(close, title="Base price")
ema1_len=input(50, title="Moving Average 1")
ema2_len=input(100, title="Moving Average 2")
ema3_len=input(200, title="Moving Average 3")

ema1 = sma(price, ema1_len)
ema2 = sma(price, ema2_len)
ema3 = sma(price, ema3_len)

p1 = plot(ema1, title="Moving Average 1", color=green)
p2 = plot(ema2, title="Moving Average 2", color=orange)
p3 = plot(ema3, title="Moving Average 3", color=red)

fill(p1, p2, color=green, transp=80)
fill(p2, p3, color=orange, transp=80)