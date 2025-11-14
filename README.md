//@version=6
strategy('Smart Money Concepts [Mkhulisi]', 'Mk - SMC', overlay = true, max_labels_count = 500, max_lines_count = 500, max_boxes_count = 500)
//---------------------------------------------------------------------------------------------------------------------}
//CONSTANTS & STRINGS & INPUTS
//---------------------------------------------------------------------------------------------------------------------{
BULLISH_LEG                     = 1
BEARISH_LEG                     = 0

BULLISH                         = +1
BEARISH                         = -1

GREEN                           = #089981
RED                             = #F23645
BLUE                            = #2157f3
GRAY                            = #878b94
MONO_BULLISH                    = #b2b5be
MONO_BEARISH                    = #5d606b

HISTORICAL                      = 'Historical'
PRESENT                         = 'Present'

COLORED                         = 'Colored'
MONOCHROME                      = 'Monochrome'

ALL                             = 'All'
BOS                             = 'BOS'
CHOCH                           = 'CHoCH'

TINY                            = size.tiny
SMALL                           = size.small
NORMAL                          = size.normal

ATR                             = 'Atr'
RANGE                           = 'Cumulative Mean Range'

CLOSE                           = 'Close'
HIGHLOW                         = 'High/Low'

SOLID                           = '⎯⎯⎯'
DASHED                          = '----'
DOTTED                          = '····'

SMART_GROUP                     = 'Smart Money Concepts'
INTERNAL_GROUP                  = 'Real Time Internal Structure'
SWING_GROUP                     = 'Real Time Swing Structure'
BLOCKS_GROUP                    = 'Order Blocks'
EQUAL_GROUP                     = 'EQH/EQL'
GAPS_GROUP                      = 'Fair Value Gaps'
LEVELS_GROUP                    = 'Highs & Lows MTF'
ZONES_GROUP                     = 'Premium & Discount Zones'
MOVING_AVERAGES_GROUP           = 'Moving Averages'
FRACTAL_GROUP                   = 'Fractals'

modeTooltip                     = 'Allows to display historical Structure or only the recent ones'
styleTooltip                    = 'Indicator color theme'
showTrendTooltip                = 'Display additional candles with a color reflecting the current trend detected by structure'
showInternalsTooltip            = 'Display internal market structure'
internalFilterConfluenceTooltip = 'Filter non significant internal structure breakouts'
showStructureTooltip            = 'Display swing market Structure'
showSwingsTooltip               = 'Display swing point as labels on the chart'
showHighLowSwingsTooltip        = 'Highlight most recent strong and weak high/low points on the chart'
showInternalOrderBlocksTooltip  = 'Display internal order blocks on the chart\n\nNumber of internal order blocks to display on the chart'
showSwingOrderBlocksTooltip     = 'Display swing order blocks on the chart\n\nNumber of internal swing blocks to display on the chart'
orderBlockFilterTooltip         = 'Method used to filter out volatile order blocks \n\nIt is recommended to use the cumulative mean range method when a low amount of data is available'
orderBlockMitigationTooltip     = 'Select what values to use for order block mitigation'
showEqualHighsLowsTooltip       = 'Display equal highs and equal lows on the chart'
equalHighsLowsLengthTooltip     = 'Number of bars used to confirm equal highs and equal lows'
equalHighsLowsThresholdTooltip  = 'Sensitivity threshold in a range (0, 1) used for the detection of equal highs & lows\n\nLower values will return fewer but more pertinent results'
showFairValueGapsTooltip        = 'Display fair values gaps on the chart'
fairValueGapsThresholdTooltip   = 'Filter out non significant fair value gaps'
fairValueGapsTimeframeTooltip   = 'Fair value gaps timeframe'
fairValueGapsExtendTooltip      = 'Determine how many bars to extend the Fair Value Gap boxes on chart'
showPremiumDiscountZonesTooltip = 'Display premium, discount, and equilibrium zones on chart'

modeInput                       = input.string( HISTORICAL, 'Mode',                     group = SMART_GROUP,    tooltip = modeTooltip, options = [HISTORICAL, PRESENT])
styleInput                      = input.string( COLORED,    'Style',                    group = SMART_GROUP,    tooltip = styleTooltip,options = [COLORED, MONOCHROME])
showTrendInput                  = input(        false,      'Color Candles',            group = SMART_GROUP,    tooltip = showTrendTooltip)

showInternalsInput              = input(        true,       'Show Internal Structure',  group = INTERNAL_GROUP, tooltip = showInternalsTooltip)
showInternalBullInput           = input.string( ALL,        'Bullish Structure',        group = INTERNAL_GROUP, inline = 'ibull', options = [ALL,BOS,CHOCH])
internalBullColorInput          = input(        GREEN,      '',                         group = INTERNAL_GROUP, inline = 'ibull')
showInternalBearInput           = input.string( ALL,        'Bearish Structure' ,       group = INTERNAL_GROUP, inline = 'ibear', options = [ALL,BOS,CHOCH])
internalBearColorInput          = input(        RED,        '',                         group = INTERNAL_GROUP, inline = 'ibear')
internalFilterConfluenceInput   = input(        false,      'Confluence Filter',        group = INTERNAL_GROUP, tooltip = internalFilterConfluenceTooltip)
internalStructureSize           = input.string( TINY,       'Internal Label Size',      group = INTERNAL_GROUP, options = [TINY,SMALL,NORMAL])

showStructureInput              = input(        true,       'Show Swing Structure',     group = SWING_GROUP,    tooltip = showStructureTooltip)
showSwingBullInput              = input.string( ALL,        'Bullish Structure',        group = SWING_GROUP,    inline = 'bull',    options = [ALL,BOS,CHOCH])
swingBullColorInput             = input(        GREEN,      '',                         group = SWING_GROUP,    inline = 'bull')
showSwingBearInput              = input.string( ALL,        'Bearish Structure',        group = SWING_GROUP,    inline = 'bear',    options = [ALL,BOS,CHOCH])
swingBearColorInput             = input(        RED,        '',                         group = SWING_GROUP,    inline = 'bear')
swingStructureSize              = input.string( SMALL,      'Swing Label Size',         group = SWING_GROUP,    options = [TINY,SMALL,NORMAL])
showSwingsInput                 = input(        false,      'Show Swings Points',       group = SWING_GROUP,    tooltip = showSwingsTooltip,inline = 'swings')
swingsLengthInput               = input.int(    50,         '',                         group = SWING_GROUP,    minval = 10,                inline = 'swings')
showHighLowSwingsInput          = input(        true,       'Show Strong/Weak High/Low',group = SWING_GROUP,    tooltip = showHighLowSwingsTooltip)

showInternalOrderBlocksInput    = input(        true,       'Internal Order Blocks' ,   group = BLOCKS_GROUP,   tooltip = showInternalOrderBlocksTooltip,   inline = 'iob')
internalOrderBlocksSizeInput    = input.int(    5,          '',                         group = BLOCKS_GROUP,   minval = 1, maxval = 20,                    inline = 'iob')
showSwingOrderBlocksInput       = input(        false,      'Swing Order Blocks',       group = BLOCKS_GROUP,   tooltip = showSwingOrderBlocksTooltip,      inline = 'ob')
swingOrderBlocksSizeInput       = input.int(    5,          '',                         group = BLOCKS_GROUP,   minval = 1, maxval = 20,                    inline = 'ob') 
orderBlockFilterInput           = input.string( 'Atr',      'Order Block Filter',       group = BLOCKS_GROUP,   tooltip = orderBlockFilterTooltip,          options = [ATR, RANGE])
orderBlockMitigationInput       = input.string( HIGHLOW,    'Order Block Mitigation',   group = BLOCKS_GROUP,   tooltip = orderBlockMitigationTooltip,      options = [CLOSE,HIGHLOW])
internalBullishOrderBlockColor  = input.color(color.new(#3179f5, 80), 'Internal Bullish OB',    group = BLOCKS_GROUP)
internalBearishOrderBlockColor  = input.color(color.new(#f77c80, 80), 'Internal Bearish OB',    group = BLOCKS_GROUP)
swingBullishOrderBlockColor     = input.color(color.new(#1848cc, 80), 'Bullish OB',             group = BLOCKS_GROUP)
swingBearishOrderBlockColor     = input.color(color.new(#b22833, 80), 'Bearish OB',             group = BLOCKS_GROUP)

showEqualHighsLowsInput         = input(        true,       'Equal High/Low',           group = EQUAL_GROUP,    tooltip = showEqualHighsLowsTooltip)
equalHighsLowsLengthInput       = input.int(    3,          'Bars Confirmation',        group = EQUAL_GROUP,    tooltip = equalHighsLowsLengthTooltip,      minval = 1)
equalHighsLowsThresholdInput    = input.float(  0.1,        'Threshold',                group = EQUAL_GROUP,    tooltip = equalHighsLowsThresholdTooltip,   minval = 0, maxval = 0.5, step = 0.1)
equalHighsLowsSizeInput         = input.string( TINY,       'Label Size',               group = EQUAL_GROUP,    options = [TINY,SMALL,NORMAL])

string fvg_group = "Fair Value Gap (FVG) - Multi-Timeframe"
bool is_enable_fvg_section = input.bool(true, title='Enable FVGs Section', group=fvg_group, inline="fvg_main", display=display.none)

// FVG Colors
color bullishFvgColor = input.color(color.new(color.green, 60), 'Bullish Color', group=fvg_group, inline="fvg_colors", display=display.none, active=is_enable_fvg_section)
color bearishFvgColor = input.color(color.new(color.red, 60), 'Bearish Color', group=fvg_group, inline="fvg_colors", display=display.none, active=is_enable_fvg_section)
color mitigatedFvgColor = input.color(color.new(#8c8c8c, 70), 'Mitigated Color', group=fvg_group, inline="fvg_colors", display=display.none, active=is_enable_fvg_section)
color fvg_border_color_bull = input.color(color.new(color.green, 100), 'Bullish Border', group=fvg_group, inline="fvg_border", display=display.none, active=is_enable_fvg_section)
color fvg_border_color_bear = input.color(color.new(color.red, 100), 'Bearish Border', group=fvg_group, inline="fvg_border", display=display.none, active=is_enable_fvg_section)

bool isMitigatedFvgToReduce = input.bool(true, title='Reduce Mitigated FVG Area', group=fvg_group, active=is_enable_fvg_section)
label_size = input.string(size.small, "FVGs Labels size", [size.tiny, size.small, size.normal, size.large, size.huge], group = fvg_group, display=display.none, active=is_enable_fvg_section)
int fvgTFDistance = input.int(10, 'FVGs Timeframe Distance', minval=1, maxval=25, group=fvg_group, tooltip="Controls how many candles on the right side between a timeframe FVG and one higher timeframe FVG.", active=is_enable_fvg_section)
bool ltf_hide = input.bool(true, "Hide FVG's lowers than enabled timeframes?", group=fvg_group, display=display.none, active=is_enable_fvg_section)
bool isAddBarIndex = input.bool(false, "Show FVG BarIndex?", group=fvg_group, display=display.none, active=is_enable_fvg_section)
int fvgHistoryNbr = input.int(8, 'Global Max Active FVGs per Timeframe', minval=1, maxval=50, group=fvg_group, tooltip="Sets the global upper limit for the number of active FVGs per timeframe. Each timeframe's Max FVGs (e.g., TF1 Max FVGs, TF2 Max FVGs) cannot exceed this value and will be capped at this limit if a higher number is entered.", active=is_enable_fvg_section)

// Warning text as pure text above fvgHistoryNbr using input.bool with display=none
//bool fvg_limit_warning_text = input.bool(false, title="All timeframe Max FVGs (e.g., TF1 Max FVGs, TF2 Max FVGs) are capped at the global upper limit below.", group=fvg_group, tooltip="This is a note: Each timeframe's Max FVGs cannot exceed the Max Active FVGs value and will be capped if higher.", display=display.none)
bool tf1_show = input.bool(true, "TF1", group = fvg_group, inline="tf1", display=display.none, active=is_enable_fvg_section)
string tf1 = input.timeframe("15", "", group=fvg_group, inline="tf1", display=display.none, active=is_enable_fvg_section)
string tf1_label = input.string("15", "", group=fvg_group, inline="tf1", display=display.none, active=is_enable_fvg_section)
int tf1_fvg_limit = input.int(6, "TF1 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf1", display=display.none, active=is_enable_fvg_section)

bool tf2_show = input.bool(true, "TF2", group = fvg_group, inline="tf2", display=display.none, active=is_enable_fvg_section)
string tf2 = input.timeframe("60", "", group=fvg_group, inline="tf2", display=display.none, active=is_enable_fvg_section)
string tf2_label = input.string("1H", "", group=fvg_group, inline="tf2", display=display.none, active=is_enable_fvg_section)
int tf2_fvg_limit = input.int(4, "TF2 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf2", display=display.none, active=is_enable_fvg_section)

bool tf3_show = input.bool(true, "TF3", group = fvg_group, inline="tf3", display=display.none, active=is_enable_fvg_section)
string tf3 = input.timeframe("240", "", group=fvg_group, inline="tf3", display=display.none, active=is_enable_fvg_section)
string tf3_label = input.string("4H", "", group=fvg_group, inline="tf3", display=display.none, active=is_enable_fvg_section)
int tf3_fvg_limit = input.int(4, "TF3 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf3", display=display.none, active=is_enable_fvg_section)

bool tf4_show = input.bool(true, "TF4", group = fvg_group, inline="tf4", display=display.none, active=is_enable_fvg_section)
string tf4 = input.timeframe("D", "", group=fvg_group, inline="tf4", display=display.none, active=is_enable_fvg_section)
string tf4_label = input.string("1D", "", group=fvg_group, inline="tf4", display=display.none, active=is_enable_fvg_section)
int tf4_fvg_limit = input.int(4, "TF4 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf4", display=display.none, active=is_enable_fvg_section)

bool tf5_show = input.bool(false, "TF5", group = fvg_group, inline="tf5", display=display.none, active=is_enable_fvg_section)
string tf5 = input.timeframe("W", "", group=fvg_group, inline="tf5", display=display.none, active=is_enable_fvg_section)
string tf5_label = input.string("1W", "", group=fvg_group, inline="tf5", display=display.none, active=is_enable_fvg_section)
int tf5_fvg_limit = input.int(4, "TF5 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf5", display=display.none, active=is_enable_fvg_section)

bool tf6_show = input.bool(false, "TF6", group = fvg_group, inline="tf6", display=display.none, active=is_enable_fvg_section)
string tf6 = input.timeframe("1M", "", group=fvg_group, inline="tf6", display=display.none, active=is_enable_fvg_section)
string tf6_label = input.string("1M", "", group=fvg_group, inline="tf6", display=display.none, active=is_enable_fvg_section)
int tf6_fvg_limit = input.int(4, "TF6 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf6", display=display.none, active=is_enable_fvg_section)

bool tf7_show = input.bool(false, "TF7", group = fvg_group, inline="tf7", display=display.none, active=is_enable_fvg_section)
string tf7 = input.timeframe("6M", "", group=fvg_group, inline="tf7", display=display.none, active=is_enable_fvg_section)
string tf7_label = input.string("6M", "", group=fvg_group, inline="tf7", display=display.none, active=is_enable_fvg_section)
int tf7_fvg_limit = input.int(4, "TF7 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf7", display=display.none, active=is_enable_fvg_section)

bool tf8_show = input.bool(false, "TF8", group = fvg_group, inline="tf8", display=display.none, active=is_enable_fvg_section)
string tf8 = input.timeframe("12M", "", group=fvg_group, inline="tf8", display=display.none, active=is_enable_fvg_section)
string tf8_label = input.string("12M", "", group=fvg_group, inline="tf8", display=display.none, active=is_enable_fvg_section)
int tf8_fvg_limit = input.int(4, "TF8 Max FVGs", minval=1, maxval=50, group=fvg_group, inline="tf8", display=display.none, active=is_enable_fvg_section)

showDailyLevelsInput            = input(        false,      'Daily',    group = LEVELS_GROUP,   inline = 'daily')
dailyLevelsStyleInput           = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'daily',   options = [SOLID,DASHED,DOTTED])
dailyLevelsColorInput           = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'daily')
showWeeklyLevelsInput           = input(        false,      'Weekly',   group = LEVELS_GROUP,   inline = 'weekly')
weeklyLevelsStyleInput          = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'weekly',  options = [SOLID,DASHED,DOTTED])
weeklyLevelsColorInput          = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'weekly')
showMonthlyLevelsInput          = input(        false,      'Monthly',  group = LEVELS_GROUP,  inline = 'monthly')
monthlyLevelsStyleInput         = input.string( SOLID,      '',         group = LEVELS_GROUP,   inline = 'monthly', options = [SOLID,DASHED,DOTTED])
monthlyLevelsColorInput         = input(        BLUE,       '',         group = LEVELS_GROUP,   inline = 'monthly')

showPremiumDiscountZonesInput   = input(        false,      'Premium/Discount Zones',    group = ZONES_GROUP , tooltip = showPremiumDiscountZonesTooltip)
premiumZoneColorInput           = input.color(  RED,        'Premium Zone',              group = ZONES_GROUP)
equilibriumZoneColorInput       = input.color(  GRAY,       'Equilibrium Zone',          group = ZONES_GROUP)
discountZoneColorInput          = input.color(  GREEN,      'Discount Zone',             group = ZONES_GROUP)

autoFibGroup = "Auto Fib Retracement"
devTooltip = "Deviation is a multiplier that affects how much the price should deviate from the previous pivot in order for the bar to become a new pivot."
depthTooltip = "The minimum number of bars that will be taken into account when calculating the indicator."

is_enable_autofib_section = input.bool(true, 'Enable Auto Fib Section', group=autoFibGroup, inline = "", display=display.none)
// pivots threshold
threshold_multiplier = input.float(title="Deviation", defval=3, minval=0, group=autoFibGroup, tooltip=devTooltip, active = is_enable_autofib_section)
depth = input.int(title="Depth", defval=10, minval=2, group=autoFibGroup, tooltip=depthTooltip, active = is_enable_autofib_section)
reverse = input(false, "Reverse", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
var extendLeft = input(false, "Extend Left    |    Extend Right", inline = "Extend Lines", group=autoFibGroup, active = is_enable_autofib_section)
var extendRight = input(true, "", inline = "Extend Lines", group=autoFibGroup, active = is_enable_autofib_section)
var extending = extend.none
if extendLeft and extendRight
    extending := extend.both
if extendLeft and not extendRight
    extending := extend.left
if not extendLeft and extendRight
    extending := extend.right
prices = input(true, "Show Prices", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
levels = input(true, "Show Levels", inline = "Levels", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
levelsFormat = input.string("Values", "", options = ["Values", "Percent"], inline = "Levels", display = display.data_window, group=autoFibGroup, active = levels and is_enable_autofib_section)
labelsPosition = input.string("Left", "Labels Position", options = ["Left", "Right"], display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
var int backgroundTransparency = input.int(85, "Background Transparency", minval = 0, maxval = 100, display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)

show_0  = input(true, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0 = input(0, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = show_0 and is_enable_autofib_section)
color_0 = input(#787b86, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = show_0 and is_enable_autofib_section)

show_0_236  = input(true, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_236 = input(0.236, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = show_0_236 and is_enable_autofib_section)
color_0_236 = input(#f44336, "", inline = "Level0", display = display.data_window, group=autoFibGroup, active = show_0_236 and is_enable_autofib_section)

show_0_382  = input(true, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_382 = input(0.382, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = show_0_382 and is_enable_autofib_section)
color_0_382 = input(#81c784, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = show_0_382 and is_enable_autofib_section)

show_0_5  = input(true, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_5 = input(0.5, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = show_0_5 and is_enable_autofib_section)
color_0_5 = input(#4caf50, "", inline = "Level1", display = display.data_window, group=autoFibGroup, active = show_0_5 and is_enable_autofib_section)

show_0_618  = input(true, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_618 = input(0.618, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = show_0_618 and is_enable_autofib_section)
color_0_618 = input(#009688, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = show_0_618 and is_enable_autofib_section)

show_0_65  = input(false, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_65 = input(0.65, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = show_0_65 and is_enable_autofib_section)
color_0_65 = input(#009688, "", inline = "Level2", display = display.data_window, group=autoFibGroup, active = show_0_65 and is_enable_autofib_section)

show_0_786  = input(true, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_0_786 = input(0.786, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = show_0_786 and is_enable_autofib_section)
color_0_786 = input(#64b5f6, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = show_0_786 and is_enable_autofib_section)

show_1  = input(true, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_1 = input(1, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = show_1 and is_enable_autofib_section)
color_1 = input(#787b86, "", inline = "Level3", display = display.data_window, group=autoFibGroup, active = show_1 and is_enable_autofib_section)

show_1_272  = input(false, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_1_272 = input(1.272, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = show_1_272 and is_enable_autofib_section)
color_1_272 = input(#81c784, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = show_1_272 and is_enable_autofib_section)

show_1_414  = input(false, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_1_414 = input(1.414, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = show_1_414 and is_enable_autofib_section)
color_1_414 = input(#f44336, "", inline = "Level4", display = display.data_window, group=autoFibGroup, active = show_1_414 and is_enable_autofib_section)

show_1_618  = input(true, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_1_618 = input(1.618, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = show_1_618 and is_enable_autofib_section)
color_1_618 = input(#2962ff, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = show_1_618 and is_enable_autofib_section)

show_1_65  = input(false, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_1_65 = input(1.65, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = show_1_65 and is_enable_autofib_section)
color_1_65 = input(#2962ff, "", inline = "Level5", display = display.data_window, group=autoFibGroup, active = show_1_65 and is_enable_autofib_section)

show_2_618  = input(true, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_2_618 = input(2.618, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = show_2_618 and is_enable_autofib_section)
color_2_618 = input(#f44336, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = show_2_618 and is_enable_autofib_section)

show_2_65  = input(false, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_2_65 = input(2.65, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = show_2_65 and is_enable_autofib_section)
color_2_65 = input(#f44336, "", inline = "Level6", display = display.data_window, group=autoFibGroup, active = show_2_65 and is_enable_autofib_section)

show_3_618  = input(true, "", inline = "Level7", display = display.data_window, group=autoFibGroup, active = is_enable_autofib_section)
value_3_618 = input(3.618, "", inline = "Level7", display = display.data_window, group=autoFibGroup, active = show_3_618 and is_enable_autofib_section)
color_3_618 = input(#9c27b0, "", inline = "Level7", display = display.data_window, group=autoFibGroup, active = show_3_618 and is_enable_autofib_section)

show_3_65  = input(false, "", inline = "Level7", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_3_65 = input(3.65,  "", inline = "Level7", display = display.data_window, group = autoFibGroup, active = show_3_65 and is_enable_autofib_section)
color_3_65 = input(#9c27b0, "", inline = "Level7", display = display.data_window, group = autoFibGroup, active = show_3_65 and is_enable_autofib_section)

show_4_236  = input(true,  "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_4_236 = input(4.236, "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = show_4_236 and is_enable_autofib_section)
color_4_236 = input(#e91e63, "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = show_4_236 and is_enable_autofib_section)

show_4_618  = input(false, "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_4_618 = input(4.618, "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = show_4_618 and is_enable_autofib_section)
color_4_618 = input(#81c784, "", inline = "Level8", display = display.data_window, group = autoFibGroup, active = show_4_618 and is_enable_autofib_section)

show_neg_0_236  = input(false, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_neg_0_236 = input(-0.236, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = show_neg_0_236 and is_enable_autofib_section)
color_neg_0_236 = input(#f44336, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = show_neg_0_236 and is_enable_autofib_section)

show_neg_0_382  = input(false, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_neg_0_382 = input(-0.382, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = show_neg_0_382 and is_enable_autofib_section)
color_neg_0_382 = input(#81c784, "", inline = "Level9", display = display.data_window, group = autoFibGroup, active = show_neg_0_382 and is_enable_autofib_section)

show_neg_0_618  = input(false, "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_neg_0_618 = input(-0.618, "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = show_neg_0_618 and is_enable_autofib_section)
color_neg_0_618 = input(#009688, "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = show_neg_0_618 and is_enable_autofib_section)

show_neg_0_65  = input(false, "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = is_enable_autofib_section)
value_neg_0_65 = input(-0.65,  "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = show_neg_0_65 and is_enable_autofib_section)
color_neg_0_65 = input(#009688, "", inline = "Level10", display = display.data_window, group = autoFibGroup, active = show_neg_0_65 and is_enable_autofib_section)

string ma_group='Moving Average - Multi-TimeFrame'
string ma_help = "Width,Transparency,LineStyle,Color"
is_enable_ma_section = input.bool(true, "Enable MA Section", group = ma_group, display=display.none, tooltip = "If you choose 1W MA, the end of the curve is at the last close of weekly. If you choose 1D MA, the end of the curve is at the last close of daily. You can find it, and it may be out of your current viewing area.")
inputTemplate = input.string(title='Template', group=ma_group, defval='DISABLED', options=['DISABLED', 'EMA 20-50-100-200', 'EMA Fibonacci 21-55-89-144-233', 'EMA 20-50-100-200 / SMA 100-200', 'EMA Golden/Death Cross 50-200', 'SMA Golden/Death Cross 50-200', 'SMA 5-10-20-50-200', 'HMA Cross 21-34','The Crypto School Long Short Trading'], tooltip = "Choose templates here to override all MA settings below.", display=display.none, active = is_enable_ma_section)
ma_setting_active = (inputTemplate == "DISABLED") and is_enable_ma_section
input_template_Timeframe = input.timeframe(title="Template Timeframe", group=ma_group, defval="", tooltip = "Use one timeframe only when using a template.", display=display.none, active = not(ma_setting_active) and is_enable_ma_section)

input_1_showMA = input.bool(true, 'MA 1 Show', group=ma_group, inline = "MA Curve Show1", display=display.none, active = is_enable_ma_section)
input_2_showMA = input.bool(true, 'MA 2 Show', group=ma_group, inline = "MA Curve Show1", display=display.none, active = is_enable_ma_section)
input_3_showMA = input.bool(true, 'MA 3 Show', group=ma_group, inline = "MA Curve Show1", display=display.none, active = is_enable_ma_section)
input_4_showMA = input.bool(true, 'MA 4 Show', group=ma_group, inline = "MA Curve Show1", display=display.none, active = is_enable_ma_section)
input_5_showMA = input.bool(true, 'MA 5 Show', group=ma_group, inline = "MA Curve Show1", display=display.none, active = is_enable_ma_section)
input_6_showMA = input.bool(true, 'MA 6 Show', group=ma_group, inline = "MA Curve Show2", display=display.none, active = is_enable_ma_section)
input_7_showMA = input.bool(true, 'MA 7 Show', group=ma_group, inline = "MA Curve Show2", display=display.none, active = is_enable_ma_section)
input_8_showMA = input.bool(true, 'MA 8 Show', group=ma_group, inline = "MA Curve Show2", display=display.none, active = is_enable_ma_section)
input_9_showMA = input.bool(true, 'MA 9 Show', group=ma_group, inline = "MA Curve Show2", display=display.none, active = is_enable_ma_section)
input_10_showMA = input.bool(true, 'MA 10 Show', group=ma_group, inline = "MA Curve Show2", display=display.none, active = is_enable_ma_section)

input_1_showLabel = input.bool(true, 'MA 1 Label', group=ma_group, inline = "MA Curve Label1", display=display.none, active = is_enable_ma_section)
input_2_showLabel = input.bool(true, 'MA 2 Label', group=ma_group, inline = "MA Curve Label1", display=display.none, active = is_enable_ma_section)
input_3_showLabel = input.bool(true, 'MA 3 Label', group=ma_group, inline = "MA Curve Label1", display=display.none, active = is_enable_ma_section)
input_4_showLabel = input.bool(true, 'MA 4 Label', group=ma_group, inline = "MA Curve Label1", display=display.none, active = is_enable_ma_section)
input_5_showLabel = input.bool(true, 'MA 5 Label', group=ma_group, inline = "MA Curve Label1", display=display.none, active = is_enable_ma_section)
input_6_showLabel = input.bool(true, 'MA 6 Label', group=ma_group, inline = "MA Curve Label2", display=display.none, active = is_enable_ma_section)
input_7_showLabel = input.bool(true, 'MA 7 Label', group=ma_group, inline = "MA Curve Label2", display=display.none, active = is_enable_ma_section)
input_8_showLabel = input.bool(true, 'MA 8 Label', group=ma_group, inline = "MA Curve Label2", display=display.none, active = is_enable_ma_section)
input_9_showLabel = input.bool(true, 'MA 9 Label', group=ma_group, inline = "MA Curve Label2", display=display.none, active = is_enable_ma_section)
input_10_showLabel = input.bool(true, 'MA 10 Label', group=ma_group, inline = "MA Curve Label2", display=display.none, active = is_enable_ma_section)
input_curve_label_color = input(title='MA Curve Label Color', group=ma_group, inline = "MA Label Style", defval=#FFFFFF, display=display.none, active = is_enable_ma_section)
input_curve_label_size = input.string(size.tiny, "MA Curve Label size", [size.tiny, size.small, size.normal, size.large, size.huge], group = ma_group, inline = "MA Label Style", display=display.none, active = is_enable_ma_section)
int input_curve_label_transp = input.int(title='MA Curve Label Transparency', group=ma_group, inline = "MA Label Style", minval=0, maxval=100, defval=30, step=5, display=display.none, active = is_enable_ma_section)

string input_1_MaType = input.string(title='MA 1', group=ma_group, inline = "MA 1", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_1_MaLength = input.int(title='', group=ma_group, inline = "MA 1", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_1_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 1", defval="", display=display.none, active = ma_setting_active)
input_1_Source = input(title='', group=ma_group, inline = "MA 1", defval=close, display=display.none, active = ma_setting_active)
int input_1_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 1a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_1_LineTransparency = input.int(title='', group=ma_group, inline = "MA 1a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_1_LineStyle = input.string(title='', group=ma_group, inline = "MA 1a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_1_color = input.color(title='', group=ma_group, inline = "MA 1a", defval=#0D47A1, display=display.none, active = ma_setting_active)

string input_2_MaType = input.string(title='MA 2', group=ma_group, inline = "MA 2", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_2_MaLength = input.int(title='', group=ma_group, inline = "MA 2", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_2_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 2", defval="", display=display.none, active = ma_setting_active)
input_2_Source = input(title='', group=ma_group, inline = "MA 2", defval=close, display=display.none, active = ma_setting_active)
int input_2_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 2a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_2_LineTransparency = input.int(title='', group=ma_group, inline = "MA 2a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_2_LineStyle = input.string(title='', group=ma_group, inline = "MA 2a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_2_color = input(title='', group=ma_group, inline = "MA 2a", defval=#B71C1C, display=display.none, active = ma_setting_active)

string input_3_MaType = input.string(title='MA 3', group=ma_group, inline = "MA 3", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_3_MaLength = input.int(title='', group=ma_group, inline = "MA 3", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_3_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 3", defval="", display=display.none, active = ma_setting_active)
input_3_Source = input(title='', group=ma_group, inline = "MA 3", defval=close, display=display.none, active = ma_setting_active)
int input_3_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 3a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_3_LineTransparency = input.int(title='', group=ma_group, inline = "MA 3a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_3_LineStyle = input.string(title='', group=ma_group, inline = "MA 3a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_3_color = input(title='', group=ma_group, inline = "MA 3a", defval=#4A148C, display=display.none, active = ma_setting_active)

string input_4_MaType = input.string(title='MA 4', group=ma_group, inline = "MA 4", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_4_MaLength = input.int(title='', group=ma_group, inline = "MA 4", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_4_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 4", defval="", display=display.none, active = ma_setting_active)
input_4_Source = input(title='', group=ma_group, inline = "MA 4", defval=close, display=display.none, active = ma_setting_active)
int input_4_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 4a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_4_LineTransparency = input.int(title='', group=ma_group, inline = "MA 4a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_4_LineStyle = input.string(title='', group=ma_group, inline = "MA 4a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_4_color = input(title='', group=ma_group, inline = "MA 4a", defval=#EEEEEE, display=display.none, active = ma_setting_active)

string input_5_MaType = input.string(title='MA 5', group=ma_group, inline = "MA 5", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_5_MaLength = input.int(title='', group=ma_group, inline = "MA 5", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_5_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 5", defval="", display=display.none, active = ma_setting_active)
input_5_Source = input(title='', group=ma_group, inline = "MA 5", defval=close, display=display.none, active = ma_setting_active)
int input_5_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 5a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_5_LineTransparency = input.int(title='', group=ma_group, inline = "MA 5a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_5_LineStyle = input.string(title='', group=ma_group, inline = "MA 5a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_5_color = input(title='', group=ma_group, inline = "MA 5a", defval=#FFCA28, display=display.none, active = ma_setting_active)

string input_6_MaType = input.string(title='MA 6', group=ma_group, inline = "MA 6", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_6_MaLength = input.int(title='', group=ma_group, inline = "MA 6", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_6_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 6", defval="", display=display.none, active = ma_setting_active)
input_6_Source = input(title='', group=ma_group, inline = "MA 6", defval=close, display=display.none, active = ma_setting_active)
int input_6_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 6a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_6_LineTransparency = input.int(title='', group=ma_group, inline = "MA 6a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_6_LineStyle = input.string(title='', group=ma_group, inline = "MA 6a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_6_color = input(title='', group=ma_group, inline = "MA 6a", defval=#1B5E20, display=display.none, active = ma_setting_active)

string input_7_MaType = input.string(title='MA 7', group=ma_group, inline = "MA 7", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_7_MaLength = input.int(title='', group=ma_group, inline = "MA 7", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_7_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 7", defval="", display=display.none, active = ma_setting_active)
input_7_Source = input(title='', group=ma_group, inline = "MA 7", defval=close, display=display.none, active = ma_setting_active)
int input_7_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 7a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_7_LineTransparency = input.int(title='', group=ma_group, inline = "MA 7a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_7_LineStyle = input.string(title='', group=ma_group, inline = "MA 7a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_7_color = input(title='', group=ma_group, inline = "MA 7a", defval=#AD1457, display=display.none, active = ma_setting_active)

string input_8_MaType = input.string(title='MA 8', group=ma_group, inline = "MA 8", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_8_MaLength = input.int(title='', group=ma_group, inline = "MA 8", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_8_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 8", defval="", display=display.none, active = ma_setting_active)
input_8_Source = input(title='', group=ma_group, inline = "MA 8", defval=close, display=display.none, active = ma_setting_active)
int input_8_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 8a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_8_LineTransparency = input.int(title='', group=ma_group, inline = "MA 8a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_8_LineStyle = input.string(title='', group=ma_group, inline = "MA 8a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_8_color = input(title='', group=ma_group, inline = "MA 8a", defval=#006064, display=display.none, active = ma_setting_active)

string input_9_MaType = input.string(title='MA 9', group=ma_group, inline = "MA 9", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_9_MaLength = input.int(title='', group=ma_group, inline = "MA 9", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_9_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 9", defval="", display=display.none, active = ma_setting_active)
input_9_Source = input(title='', group=ma_group, inline = "MA 9", defval=close, display=display.none, active = ma_setting_active)
int input_9_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 9a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_9_LineTransparency = input.int(title='', group=ma_group, inline = "MA 9a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_9_LineStyle = input.string(title='', group=ma_group, inline = "MA 9a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_9_color = input(title='', group=ma_group, inline = "MA 9a", defval=#1DE9B6, display=display.none, active = ma_setting_active)

string input_10_MaType = input.string(title='MA10', group=ma_group, inline = "MA 10", defval='DISABLED', options=['DISABLED', 'COVWMA', 'DEMA', 'EMA', 'EHMA', 'FRAMA', 'HMA', 'KAMA', 'SMA', 'SMMA (RMA)', 'TEMA', 'VIDYA', 'VWMA', 'WMA'], display=display.none, active = ma_setting_active)
int input_10_MaLength = input.int(title='', group=ma_group, inline = "MA 10", minval=1, maxval=1000, defval=20, display=display.none, active = ma_setting_active)
input_10_Timeframe = input.timeframe(title="", group=ma_group, inline = "MA 10", defval="", display=display.none, active = ma_setting_active)
input_10_Source = input(title='', group=ma_group, inline = "MA 10", defval=close, display=display.none, active = ma_setting_active)
int input_10_LineWidth = input.int(title='Style:', group=ma_group, inline = "MA 10a", minval=1, maxval=4, defval=2, step=1, display=display.none, active = ma_setting_active)
int input_10_LineTransparency = input.int(title='', group=ma_group, inline = "MA 10a", minval=0, maxval=100, defval=0, step=5, display=display.none, active = ma_setting_active)
string input_10_LineStyle = input.string(title='', group=ma_group, inline = "MA 10a", defval='Line', options=['Line', 'Stepline', 'Step Line With Diamonds', 'Circles'], tooltip = ma_help, display=display.none, active = ma_setting_active)
color input_10_color = input(title='', group=ma_group, inline = "MA 10a", defval=#00E5FF, display=display.none, active = ma_setting_active)

showFractalsInput               = input(        true,       'Show Fractal Structure',    group = FRACTAL_GROUP)
showBullishFractalsInput        = input(        true,       'Show Bullish Fractals',     group = FRACTAL_GROUP, inline = 'bull_frac')
fractalBullishColorInput        = input.color(  GREEN,      '',                          group = FRACTAL_GROUP, inline = 'bull_frac')
showBearishFractalsInput        = input(        true,       'Show Bearish Fractals',     group = FRACTAL_GROUP, inline = 'bear_frac')
fractalBearishColorInput        = input.color(  RED,        '',                          group = FRACTAL_GROUP, inline = 'bear_frac')
fractalLookbackInput            = input.int(    1,          'Confirmation Bars',         group = FRACTAL_GROUP, minval = 1, maxval = 5, tooltip = 'Number of bars required to confirm a fractal')
fractalLineWidthInput           = input.int(    1,          'Structure Line Width',      group = FRACTAL_GROUP, minval = 1, maxval = 5)
showFractalLabelsInput          = input(        true,       'Show Fractal Labels',       group = FRACTAL_GROUP)
highlightStructureChangeInput   = input(        false,      'Highlight Structure Change',group = FRACTAL_GROUP, tooltip = 'Add background color when structure changes')

//---------------------------------------------------------------------------------------------------------------------}
//DATA STRUCTURES & VARIABLES
//---------------------------------------------------------------------------------------------------------------------{
// @type                            UDT representing alerts as bool fields
// @field internalBullishBOS        internal structure custom alert
// @field internalBearishBOS        internal structure custom alert
// @field internalBullishCHoCH      internal structure custom alert
// @field internalBearishCHoCH      internal structure custom alert
// @field swingBullishBOS           swing structure custom alert
// @field swingBearishBOS           swing structure custom alert
// @field swingBullishCHoCH         swing structure custom alert
// @field swingBearishCHoCH         swing structure custom alert
// @field internalBullishOrderBlock internal order block custom alert
// @field internalBearishOrderBlock internal order block custom alert
// @field swingBullishOrderBlock    swing order block custom alert
// @field swingBearishOrderBlock    swing order block custom alert
// @field equalHighs                equal high low custom alert
// @field equalLows                 equal high low custom alert
// @field bullishFairValueGap       fair value gap custom alert
// @field bearishFairValueGap       fair value gap custom alert
type colorSet
    color c1
    color c2
    color c3
    color c4
    color c5
    color c6
    color c7
    color c8
    color c9
    color c10

type alerts
    bool internalBullishBOS         = false
    bool internalBearishBOS         = false
    bool internalBullishCHoCH       = false
    bool internalBearishCHoCH       = false
    bool swingBullishBOS            = false
    bool swingBearishBOS            = false
    bool swingBullishCHoCH          = false
    bool swingBearishCHoCH          = false
    bool internalBullishOrderBlock  = false
    bool internalBearishOrderBlock  = false
    bool swingBullishOrderBlock     = false
    bool swingBearishOrderBlock     = false
    bool equalHighs                 = false
    bool equalLows                  = false
    bool bullishFairValueGap        = false
    bool bearishFairValueGap        = false

// @type                            UDT representing last swing extremes (top & bottom)
// @field top                       last top swing price
// @field bottom                    last bottom swing price
// @field barTime                   last swing bar time
// @field barIndex                  last swing bar index
// @field lastTopTime               last top swing time
// @field lastBottomTime            last bottom swing time
type trailingExtremes
    float top
    float bottom
    int barTime
    int barIndex
    int lastTopTime
    int lastBottomTime

// @type                            UDT representing Fair Value Gaps
// @field top                       top price
// @field bottom                    bottom price
// @field bias                      bias (BULLISH or BEARISH)
// @field topBox                    top box
// @field bottomBox                 bottom box
type Helper
    string name         = "Helper"

type FVG
    int tf_idx
    string tf
    string tf_label
    int tf_num
    box _box
    bool _type
    bool _isMitigated
    bool _isDrawn
    int left
    float top
    int right
    float bottom
    string extend=extend.none
    string border_style=line.style_solid
    int border_width=1
    color bgcolor
    color border_color
    string _text
    color text_color=color.white
    string text_halign=text.align_right
    string text_size

// @type                            UDT representing trend bias
// @field bias                      BULLISH or BEARISH
type trend
    int bias    

// @type                            UDT representing Equal Highs Lows display
// @field l_ine                     displayed line
// @field l_abel                    displayed label
type equalDisplay
    line l_ine      = na
    label l_abel    = na

// @type                            UDT representing a pivot point (swing point) 
// @field currentLevel              current price level
// @field lastLevel                 last price level
// @field crossed                   true if price level is crossed
// @field barTime                   bar time
// @field barIndex                  bar index    
type pivot
    float currentLevel
    float lastLevel
    bool crossed
    int barTime     = time
    int barIndex    = bar_index

// @type                            UDT representing an order block
// @field barHigh                   bar high
// @field barLow                    bar low
// @field barTime                   bar time
// @field bias                      BULLISH or BEARISH
type orderBlock
    float barHigh
    float barLow
    int barTime    
    int bias

// @type                            UDT representing a fractal point
// @field price                     price level of fractal
// @field barTime                   bar time
// @field barIndex                  bar index
// @field isBullish                 true for fractal low, false for fractal high
// @field confirmed                 true when fractal is confirmed
type fractalPoint
    float price
    int barTime
    int barIndex
    bool isBullish
    bool confirmed = false

// @variable                        current swing pivot high    
var pivot swingHigh                 = pivot.new(na,na,false)
// @variable                        current swing pivot low
var pivot swingLow                  = pivot.new(na,na,false)
// @variable                        current internal pivot high
var pivot internalHigh              = pivot.new(na,na,false)
// @variable                        current internal pivot low
var pivot internalLow               = pivot.new(na,na,false)
// @variable                        current equal high pivot
var pivot equalHigh                 = pivot.new(na,na,false)
// @variable                        current equal low pivot
var pivot equalLow                  = pivot.new(na,na,false)
// @variable                        swing trend bias
var trend swingTrend                = trend.new(0)
// @variable                        internal trend bias
var trend internalTrend             = trend.new(0)
// @variable                        equal high display
var equalDisplay equalHighDisplay   = equalDisplay.new()
// @variable                        equal low display
var equalDisplay equalLowDisplay    = equalDisplay.new()
// @variable                        storage for fairValueGap UDTs
var FVG[] fvgBox1 = array.new<FVG>()
var FVG[] fvgBox2 = array.new<FVG>()
var FVG[] fvgBox3 = array.new<FVG>()
var FVG[] fvgBox4 = array.new<FVG>()
var FVG[] fvgBox5 = array.new<FVG>()
var FVG[] fvgBox6 = array.new<FVG>()
var FVG[] fvgBox7 = array.new<FVG>()
var FVG[] fvgBox8 = array.new<FVG>()
// @variable                        storage for parsed highs
var array<float> parsedHighs        = array.new<float>()
// @variable                        storage for parsed lows
var array<float> parsedLows         = array.new<float>()
// @variable                        storage for raw highs
var array<float> highs              = array.new<float>()
// @variable                        storage for raw lows
var array<float> lows               = array.new<float>()
// @variable                        storage for bar time values
var array<int> times                = array.new<int>()
// @variable                        last trailing swing high and low
var trailingExtremes trailing       = trailingExtremes.new()
// @variable                                storage for orderBlock UDTs (swing order blocks)
var array<orderBlock> swingOrderBlocks      = array.new<orderBlock>()
// @variable                                storage for orderBlock UDTs (internal order blocks)
var array<orderBlock> internalOrderBlocks   = array.new<orderBlock>()
// @variable                                storage for swing order blocks boxes
var array<box> swingOrderBlocksBoxes        = array.new<box>()
// @variable                                storage for internal order blocks boxes
var array<box> internalOrderBlocksBoxes     = array.new<box>()
// @variable                        storage for fractal points
var array<fractalPoint> fractalHighs        = array.new<fractalPoint>()
// @variable                        storage for fractal points
var array<fractalPoint> fractalLows         = array.new<fractalPoint>()
// @variable                        current fractal structure bias
var int fractalStructureBias                = 0
// @variable                        previous fractal structure bias
var int prevFractalStructureBias            = 0
// @variable                        color for swing bullish structures
var swingBullishColor               = styleInput == MONOCHROME ? MONO_BULLISH : swingBullColorInput
// @variable                        color for swing bearish structures
var swingBearishColor               = styleInput == MONOCHROME ? MONO_BEARISH : swingBearColorInput
// @variable                        color for premium zone
var premiumZoneColor                = styleInput == MONOCHROME ? MONO_BEARISH : premiumZoneColorInput
// @variable                        color for discount zone
var discountZoneColor               = styleInput == MONOCHROME ? MONO_BULLISH : discountZoneColorInput 
// @variable                        bar index on current script iteration
varip int currentBarIndex           = bar_index
// @variable                        bar index on last script iteration
varip int lastBarIndex              = bar_index
// @variable                        alerts in current bar
alerts currentAlerts                = alerts.new()
// @variable                        time at start of chart
var initialTime                     = time

// we create the needed boxes for displaying order blocks at the first execution
if barstate.isfirst
    if showSwingOrderBlocksInput
        for index = 1 to swingOrderBlocksSizeInput
            swingOrderBlocksBoxes.push(box.new(na,na,na,na,xloc = xloc.bar_time,extend = extend.right))
    if showInternalOrderBlocksInput
        for index = 1 to internalOrderBlocksSizeInput
            internalOrderBlocksBoxes.push(box.new(na,na,na,na,xloc = xloc.bar_time,extend = extend.right))

// @variable                        source to use in bearish order blocks mitigation
bearishOrderBlockMitigationSource   = orderBlockMitigationInput == CLOSE ? close : high
// @variable                        source to use in bullish order blocks mitigation
bullishOrderBlockMitigationSource   = orderBlockMitigationInput == CLOSE ? close : low
// @variable                        default volatility measure
atrMeasure                          = ta.atr(200)
// @variable                        parsed volatility measure by user settings
volatilityMeasure                   = orderBlockFilterInput == ATR ? atrMeasure : ta.cum(ta.tr)/bar_index
// @variable                        true if current bar is a high volatility bar
highVolatilityBar                   = (high - low) >= (2 * volatilityMeasure)
// @variable                        parsed high
parsedHigh                          = highVolatilityBar ? low : high
// @variable                        parsed low
parsedLow                           = highVolatilityBar ? high : low

// we store current values into the arrays at each bar
parsedHighs.push(parsedHigh)
parsedLows.push(parsedLow)
highs.push(high)
lows.push(low)
times.push(time)

//---------------------------------------------------------------------------------------------------------------------}
//USER-DEFINED FUNCTIONS
//---------------------------------------------------------------------------------------------------------------------{
import TradingView/ZigZag/7 as zigzag

update()=>
    var zigzag.Pivot lastP1 = na
    var float startPrice1 = na
    var float height1 = na
    if is_enable_autofib_section
        var settings = zigzag.Settings.new(threshold_multiplier, depth, color(na), false, false, false, false, "Absolute", true)
        var zigzag.ZigZag zigZag = zigzag.newInstance(settings)
        if barstate.islast and zigZag.pivots.size() < 2
            runtime.error("Not enough data to calculate Auto Fib Retracement on the current symbol. Change the chart's timeframe to a lower one or select a smaller calculation depth using the indicator's `Depth` settings.")
        settings.devThreshold := ta.atr(10) / close * 100 * threshold_multiplier
        if zigZag.update()
            lastP1 := zigZag.lastPivot()
            if not na(lastP1)
                var line lineLast = na
                if na(lineLast)
                    lineLast := line.new(lastP1.start, lastP1.end, xloc=xloc.bar_time, color=color.gray, width=1, style=line.style_dashed)
                else
                    line.set_first_point(lineLast, lastP1.start)
                    line.set_second_point(lineLast, lastP1.end)

                startPrice1 := reverse ? lastP1.start.price : lastP1.end.price
                endPrice = reverse ? lastP1.end.price : lastP1.start.price
                height1 := (startPrice1 > endPrice ? -1 : 1) * math.abs(startPrice1 - endPrice)
    [lastP1, startPrice1, height1]

_draw_line(price, col) =>
    var id = line.new(lastP.start.time, lastP.start.price, time, price, xloc=xloc.bar_time, color=col, width=2, extend=extending)
    line.set_xy1(id, lastP.start.time, price)
    line.set_xy2(id, lastP.end.time, price)
    id

_draw_label(price, txt, txtColor) =>
    x = labelsPosition == "Left" ? lastP.start.time : not extendRight ? lastP.end.time : time
    labelStyle = labelsPosition == "Left" ? label.style_label_right : label.style_label_left
    align = labelsPosition == "Left" ? text.align_right : text.align_left
    labelsAlignStrLeft = txt + '\n ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏ \n'
    labelsAlignStrRight = '       ' + txt + '\n ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏ \n'
    labelsAlignStr = labelsPosition == "Left" ? labelsAlignStrLeft : labelsAlignStrRight
    var id = label.new(x=x, y=price, xloc=xloc.bar_time, text=labelsAlignStr, textcolor=txtColor, style=labelStyle, textalign=align, color=#00000000)
    label.set_xy(id, x, price)
    label.set_text(id, labelsAlignStr)
    label.set_textcolor(id, txtColor)

_wrap(txt) =>
    "(" + str.tostring(txt, format.mintick) + ")"

_label_txt(level, price) =>
    l = levelsFormat == "Values" ? str.tostring(level) : str.tostring(level * 100) + "%"
    (levels ? l : "") + (prices ? _wrap(price) : "")

_crossing_level(series float sr, series float r) =>
    (r > sr and r < sr[1]) or (r < sr and r > sr[1])

processLevel(bool show, float value, color colorL, line lineIdOther) =>
    float m = value
    r = startPrice + height * m
    crossed = _crossing_level(close, r)
    if show and not na(lastP)
        lineId = _draw_line(r, colorL)
        _draw_label(r, _label_txt(m, r), colorL)
        if crossed
            alert("Autofib: " + syminfo.ticker + " crossing level " + str.tostring(value))
        if not na(lineIdOther)
            linefill.new(lineId, lineIdOther, color = color.new(colorL, backgroundTransparency))
        lineId
    else
        lineIdOther
// @function            Get the value of the current leg, it can be 0 (bearish) or 1 (bullish)
// @returns             int
// === Human Readable Timeframe Converter (Fixed for v6) ===
f_timeframeToLabel(tf) =>
    // Handle special cases first
    if tf == "D"
        "1D"
    else if tf == "W"
        "1W"
    else if tf == "M"
        "1M"
    else
        // Try to parse numeric values
        tfNum = str.tonumber(tf)
        if na(tfNum)
            tf // Return as-is if not numeric
        else if tfNum % 60 == 0
            str.tostring(tfNum / 60) + "H" // Convert minutes to hours
        else
            str.tostring(tfNum) // Keep minutes as-is

// scale to the range of [min, max]
_funcMinMax(X, p, min, max) =>
    hi = ta.highest(X, p)
    lo = ta.lowest(X, p)
    (max - min) * (X - lo) / (hi - lo) + min

_funcLineStyle(_type) =>
    switch _type
        "Circles" => plot.style_circles
        "Stepline" => plot.style_stepline
        "Line With Breaks" => plot.style_linebr
        "Step Line With Diamonds" => plot.style_stepline_diamond
        =>
            plot.style_line

_funcSetFinalVal(_tpl, _type_1, _type_2, _type_3, _type_4, _type_5, _type_6, _type_7, _type_8, _maTypeDefault, _length_1, _length_2, _length_3, _length_4, _length_5, _length_6, _length_7, _length_8, _maLengthDefault) =>
    string _tmpMaType = _maTypeDefault
    int _tmpMaLength = _maLengthDefault
    if _tpl == 'EMA 20-50-100-200'
        _tmpMaType := _type_1
        _tmpMaLength := _length_1
        _tmpMaLength
    else if _tpl == 'SMA 5-10-20-50-200'
        _tmpMaType := _type_2
        _tmpMaLength := _length_2
        _tmpMaLength
    else if _tpl == 'EMA Fibonacci 21-55-89-144-233'
        _tmpMaType := _type_3
        _tmpMaLength := _length_3
        _tmpMaLength
    else if _tpl == 'EMA 20-50-100-200 / SMA 100-200'
        _tmpMaType := _type_4
        _tmpMaLength := _length_4
        _tmpMaLength
    else if _tpl == 'EMA Golden/Death Cross 50-200'
        _tmpMaType := _type_5
        _tmpMaLength := _length_5
        _tmpMaLength
    else if _tpl == 'SMA Golden/Death Cross 50-200'
        _tmpMaType := _type_6
        _tmpMaLength := _length_6
        _tmpMaLength
    else if _tpl == 'HMA Cross 21-34'
        _tmpMaType := _type_7
        _tmpMaLength := _length_7
        _tmpMaLength
    else if _tpl == 'The Crypto School Long Short Trading'
        _tmpMaType := _type_8
        _tmpMaLength := _length_8
        _tmpMaLength
    [_tmpMaType, _tmpMaLength]

// Chande Momentum Oscillator
_funcCMO(_src, _period) =>
    tmpMomentum = ta.change(_src)
    tmpSummaryUp = math.sum(math.max(tmpMomentum, 0), _period)
    tmpSummaryDown = math.sum(-math.min(tmpMomentum, 0), _period)
    math.abs((tmpSummaryUp - tmpSummaryDown) / (tmpSummaryUp + tmpSummaryDown))

// Triple EMA (TEMA)
_funcTEMA(_src, _period) =>
    ema1 = ta.ema(_src, _period)
    ema2 = ta.ema(ema1, _period)
    ema3 = ta.ema(ema2, _period)
    3 * (ema1 - ema2) + ema3

// Double Expotential Moving Average (DEMA)
_funcDEMA(_src, _period) =>
    2 * ta.ema(_src, _period) - ta.ema(ta.ema(_src, _period), _period)

// Exponential Hull Moving Average (EHMA)
_funcEHMA(_src, _period) =>
    tmpVal = math.max(2, _period)
    ta.ema(2 * ta.ema(_src, tmpVal / 2) - ta.ema(_src, tmpVal), math.round(math.sqrt(tmpVal)))

// Kaufman's Adaptive Moving Average (KAMA)
_funcKAMA(_src, _period) =>
    tmpVal = 0.0
    sum_1 = math.sum(math.abs(_src - _src[1]), _period)
    sum_2 = math.sum(math.abs(_src - _src[1]), _period)
    tmpVal := nz(tmpVal[1]) + math.pow((sum_1 != 0 ? math.abs(_src - _src[_period]) / sum_2 : 0) * (0.666 - 0.0645) + 0.0645, 2) * (_src - nz(tmpVal[1]))
    tmpVal

// Variable Index Dynamic Average (VIDYA)
_funcVIDYA(_src, _period) =>
    cmo = _funcCMO(_src, _period)
    alpha = 2 / (_period + 1)
    vidya = 0.0
    vidya := _src * alpha * cmo + nz(vidya[1]) * (1 - alpha * cmo)
    vidya

// Coefficient of Variation Weighted Moving Average (COVWMA)
_funcCOVWMA(_src, _period) =>
    _coefficientOfVariation = ta.stdev(_src, _period) / ta.sma(_src, _period)
    _cw = _src * _coefficientOfVariation
    math.sum(_cw, _period) / math.sum(_coefficientOfVariation, _period)

// Fractal Adaptive Moving Average (FRAMA)
_funcFRAMA(_src, _period) =>
    _coefficient = -4.6
    _n3 = (ta.highest(high, _period) - ta.lowest(low, _period)) / _period
    _hd2 = ta.highest(high, _period / 2)
    _ld2 = ta.lowest(low, _period / 2)
    _n2 = (_hd2 - _ld2) / (_period / 2)
    _n1 = (_hd2[_period / 2] - _ld2[_period / 2]) / (_period / 2)
    _dim = _n1 > 0 and _n2 > 0 and _n3 > 0 ? (math.log(_n1 + _n2) - math.log(_n3)) / math.log(2) : 0
    _alpha = math.exp(_coefficient * (_dim - 1))
    _sc = _alpha < 0.01 ? 0.01 : _alpha > 1 ? 1 : _alpha
    ta.cum(1) <= 2 * _period ? _src : _src * _sc + nz(_src[1]) * (1 - _sc)

// [已修复缩进错误 + Gaps]
// calculate the moving average with the transferred settings
_funcCalcMA(_type, _src, _timeframe, _period) =>
    float ma = switch _type
        "COVWMA" => _funcCOVWMA(_src, _period)
        "DEMA" => _funcDEMA(_src, _period)
        "EMA" => ta.ema(_src, _period)
        "EHMA" => _funcEHMA(_src, _period)
        "HMA" => ta.hma(_src, _period)
        "KAMA" => _funcKAMA(_src, _period)
        "SMA" => ta.sma(_src, _period) // <--- 修复了原始的缩进错误
        "SMMA (RMA)" => ta.rma(_src, _period)
        "TEMA" => _funcTEMA(_src, _period)
        "FRAMA" => _funcFRAMA(_src, _period)
        "VWMA" => ta.vwma(_src, _period)
        "VIDYA" => _funcVIDYA(_src, _period)
        "WMA" => ta.wma(_src, _period)
        =>
            float(na)
    // [关键修复] 添加 gaps=barmerge.gaps_on.
    // 这将使线条显示为阶梯状, 匹配 MA Ribbon 的行为
    request.security(syminfo.tickerid, _timeframe, ma, gaps = barmerge.gaps_on)

leg(int size) =>
    var leg     = 0    
    newLegHigh  = high[size] > ta.highest( size)
    newLegLow   = low[size]  < ta.lowest(  size)
    
    if newLegHigh
        leg := BEARISH_LEG
    else if newLegLow
        leg := BULLISH_LEG
    leg

// @function            Identify whether the current value is the start of a new leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfNewLeg(int leg)      => ta.change(leg) != 0

// @function            Identify whether the current level is the start of a new bearish leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfBearishLeg(int leg)  => ta.change(leg) == -1

// @function            Identify whether the current level is the start of a new bullish leg (swing)
// @param leg           (int) Current leg value
// @returns             bool
startOfBullishLeg(int leg)  => ta.change(leg) == +1

// @function            create a new label
// @param labelTime     bar time coordinate
// @param labelPrice    price coordinate
// @param tag           text to display
// @param labelColor    text color
// @param labelStyle    label style
// @returns             label ID
drawLabel(int labelTime, float labelPrice, string tag, color labelColor, string labelStyle) =>    
    var label l_abel = na

    if modeInput == PRESENT
        l_abel.delete()

    l_abel := label.new(chart.point.new(labelTime,na,labelPrice),tag,xloc.bar_time,color=color(na),textcolor=labelColor,style = labelStyle,size = size.small)

// @function            create a new line and label representing an EQH or EQL
// @param p_ivot        starting pivot
// @param level         price level of current pivot
// @param size          how many bars ago was the current pivot detected
// @param equalHigh     true for EQH, false for EQL
// @returns             label ID
drawEqualHighLow(pivot p_ivot, float level, int size, bool equalHigh) =>
    equalDisplay e_qualDisplay = equalHigh ? equalHighDisplay : equalLowDisplay
    
    string tag          = 'EQL'
    color equalColor    = swingBullishColor
    string labelStyle   = label.style_label_up

    if equalHigh
        tag         := 'EQH'
        equalColor  := swingBearishColor
        labelStyle  := label.style_label_down

    if modeInput == PRESENT
        line.delete(    e_qualDisplay.l_ine)
        label.delete(   e_qualDisplay.l_abel)
        
    e_qualDisplay.l_ine     := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time[size],na,level), xloc = xloc.bar_time, color = equalColor, style = line.style_dotted)
    labelPosition           = math.round(0.5*(p_ivot.barIndex + bar_index - size))
    e_qualDisplay.l_abel    := label.new(chart.point.new(na,labelPosition,level), tag, xloc.bar_index, color = color(na), textcolor = equalColor, style = labelStyle, size = equalHighsLowsSizeInput)

// @function            store current structure and trailing swing points, and also display swing points and equal highs/lows
// @param size          (int) structure size
// @param equalHighLow  (bool) true for displaying current highs/lows
// @param internal      (bool) true for getting internal structures
// @returns             label ID
getCurrentStructure(int size,bool equalHighLow = false, bool internal = false) =>        
    currentLeg              = leg(size)
    newPivot                = startOfNewLeg(currentLeg)
    pivotLow                = startOfBullishLeg(currentLeg)
    pivotHigh               = startOfBearishLeg(currentLeg)

    if newPivot
        if pivotLow
            pivot p_ivot    = equalHighLow ? equalLow : internal ? internalLow : swingLow    

            if equalHighLow and math.abs(p_ivot.currentLevel - low[size]) < equalHighsLowsThresholdInput * atrMeasure                
                drawEqualHighLow(p_ivot, low[size], size, false)

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := low[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.bottom         := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastBottomTime := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel < p_ivot.lastLevel ? 'LL' : 'HL', swingBullishColor, label.style_label_up)            
        else
            pivot p_ivot = equalHighLow ? equalHigh : internal ? internalHigh : swingHigh

            if equalHighLow and math.abs(p_ivot.currentLevel - high[size]) < equalHighsLowsThresholdInput * atrMeasure
                drawEqualHighLow(p_ivot,high[size],size,true)                

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := high[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.top            := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastTopTime    := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel > p_ivot.lastLevel ? 'HH' : 'LH', swingBearishColor, label.style_label_down)
                
// @function                draw line and label representing a structure
// @param p_ivot            base pivot point
// @param tag               test to display
// @param structureColor    base color
// @param lineStyle         line style
// @param labelStyle        label style
// @param labelSize         text size
// @returns                 label ID
drawStructure(pivot p_ivot, string tag, color structureColor, string lineStyle, string labelStyle, string labelSize) =>    
    var line l_ine      = line.new(na,na,na,na,xloc = xloc.bar_time)
    var label l_abel    = label.new(na,na)

    if modeInput == PRESENT
        l_ine.delete()
        l_abel.delete()

    l_ine   := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time,na,p_ivot.currentLevel), xloc.bar_time, color=structureColor, style=lineStyle)
    l_abel  := label.new(chart.point.new(na,math.round(0.5*(p_ivot.barIndex+bar_index)),p_ivot.currentLevel), tag, xloc.bar_index, color=color(na), textcolor=structureColor, style=labelStyle, size = labelSize)

// @function            delete order blocks
// @param internal      true for internal order blocks
// @returns             orderBlock ID
deleteOrderBlocks(bool internal = false) =>
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks

    for [index,eachOrderBlock] in orderBlocks
        bool crossedOderBlock = false
        
        if bearishOrderBlockMitigationSource > eachOrderBlock.barHigh and eachOrderBlock.bias == BEARISH
            crossedOderBlock := true
            if internal
                currentAlerts.internalBearishOrderBlock := true
            else
                currentAlerts.swingBearishOrderBlock    := true
        else if bullishOrderBlockMitigationSource < eachOrderBlock.barLow and eachOrderBlock.bias == BULLISH
            crossedOderBlock := true
            if internal
                currentAlerts.internalBullishOrderBlock := true
            else
                currentAlerts.swingBullishOrderBlock    := true
        if crossedOderBlock                    
            orderBlocks.remove(index)            

// @function            fetch and store order blocks
// @param p_ivot        base pivot point
// @param internal      true for internal order blocks
// @param bias          BULLISH or BEARISH
// @returns             void
storeOrdeBlock(pivot p_ivot,bool internal = false,int bias) =>
    if (not internal and showSwingOrderBlocksInput) or (internal and showInternalOrderBlocksInput)

        array<float> a_rray = na
        int parsedIndex = na

        if bias == BEARISH
            a_rray      := parsedHighs.slice(p_ivot.barIndex,bar_index)
            parsedIndex := p_ivot.barIndex + a_rray.indexof(a_rray.max())  
        else
            a_rray      := parsedLows.slice(p_ivot.barIndex,bar_index)
            parsedIndex := p_ivot.barIndex + a_rray.indexof(a_rray.min())                        

        orderBlock o_rderBlock          = orderBlock.new(parsedHighs.get(parsedIndex), parsedLows.get(parsedIndex), times.get(parsedIndex),bias)
        array<orderBlock> orderBlocks   = internal ? internalOrderBlocks : swingOrderBlocks
        
        if orderBlocks.size() >= 100
            orderBlocks.pop()
        orderBlocks.unshift(o_rderBlock)

// @function            draw order blocks as boxes
// @param internal      true for internal order blocks
// @returns             void
drawOrderBlocks(bool internal = false) =>        
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks
    orderBlocksSize = orderBlocks.size()

    if orderBlocksSize > 0        
        maxOrderBlocks                      = internal ? internalOrderBlocksSizeInput : swingOrderBlocksSizeInput
        array<orderBlock> parsedOrdeBlocks  = orderBlocks.slice(0, math.min(maxOrderBlocks,orderBlocksSize))
        array<box> b_oxes                   = internal ? internalOrderBlocksBoxes : swingOrderBlocksBoxes        

        for [index,eachOrderBlock] in parsedOrdeBlocks
            orderBlockColor = styleInput == MONOCHROME ? (eachOrderBlock.bias == BEARISH ? color.new(MONO_BEARISH,80) : color.new(MONO_BULLISH,80)) : internal ? (eachOrderBlock.bias == BEARISH ? internalBearishOrderBlockColor : internalBullishOrderBlockColor) : (eachOrderBlock.bias == BEARISH ? swingBearishOrderBlockColor : swingBullishOrderBlockColor)

            box b_ox        = b_oxes.get(index)
            b_ox.set_top_left_point(    chart.point.new(eachOrderBlock.barTime,na,eachOrderBlock.barHigh))
            b_ox.set_bottom_right_point(chart.point.new(last_bar_time,na,eachOrderBlock.barLow))        
            b_ox.set_border_color(      internal ? na : orderBlockColor)
            b_ox.set_bgcolor(           orderBlockColor)

// @function            detect and draw structures, also detect and store order blocks
// @param internal      true for internal structures or order blocks
// @returns             void
displayStructure(bool internal = false) =>
    var bullishBar = true
    var bearishBar = true

    if internalFilterConfluenceInput
        bullishBar := high - math.max(close, open) > math.min(close, open - low)
        bearishBar := high - math.max(close, open) < math.min(close, open - low)
    
    pivot p_ivot    = internal ? internalHigh : swingHigh
    trend t_rend    = internal ? internalTrend : swingTrend

    lineStyle       = internal ? line.style_dashed : line.style_solid
    labelSize       = internal ? internalStructureSize : swingStructureSize

    extraCondition  = internal ? internalHigh.currentLevel != swingHigh.currentLevel and bullishBar : true
    bullishColor    = styleInput == MONOCHROME ? MONO_BULLISH : internal ? internalBullColorInput : swingBullColorInput

    if ta.crossover(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BEARISH ? CHOCH : BOS

        if internal
            currentAlerts.internalBullishCHoCH  := tag == CHOCH
            currentAlerts.internalBullishBOS    := tag == BOS
        else
            currentAlerts.swingBullishCHoCH     := tag == CHOCH
            currentAlerts.swingBullishBOS       := tag == BOS

        p_ivot.crossed  := true
        t_rend.bias     := BULLISH

        displayCondition = internal ? showInternalsInput and (showInternalBullInput == ALL or (showInternalBullInput == BOS and tag != CHOCH) or (showInternalBullInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBullInput == ALL or (showSwingBullInput == BOS and tag != CHOCH) or (showSwingBullInput == CHOCH and tag == CHOCH))

        if displayCondition                        
            drawStructure(p_ivot,tag,bullishColor,lineStyle,label.style_label_down,labelSize)

        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BULLISH)

    p_ivot          := internal ? internalLow : swingLow    
    extraCondition  := internal ? internalLow.currentLevel != swingLow.currentLevel and bearishBar : true
    bearishColor    = styleInput == MONOCHROME ? MONO_BEARISH : internal ? internalBearColorInput : swingBearColorInput

    if ta.crossunder(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BULLISH ? CHOCH : BOS

        if internal
            currentAlerts.internalBearishCHoCH  := tag == CHOCH
            currentAlerts.internalBearishBOS    := tag == BOS
        else
            currentAlerts.swingBearishCHoCH     := tag == CHOCH
            currentAlerts.swingBearishBOS       := tag == BOS

        p_ivot.crossed := true
        t_rend.bias := BEARISH

        displayCondition = internal ? showInternalsInput and (showInternalBearInput == ALL or (showInternalBearInput == BOS and tag != CHOCH) or (showInternalBearInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBearInput == ALL or (showSwingBearInput == BOS and tag != CHOCH) or (showSwingBearInput == CHOCH and tag == CHOCH))
        
        if displayCondition                        
            drawStructure(p_ivot,tag,bearishColor,lineStyle,label.style_label_up,labelSize)

        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BEARISH)

// @function            draw one fair value gap box (each fair value gap has two boxes)
// @param leftTime      left time coordinate
// @param rightTime     right time coordinate
// @param topPrice      top price level
// @param bottomPrice   bottom price level
// @param boxColor      box color
// @returns             box ID
findFVGs (int tf_idx, string tf, string tf_label, int tf_num, FVG[] fvgBox) =>
    if (not ltf_hide or (ltf_hide and helper.Validtimeframe(tf)))
        [htf_high1, htf_low1, htf_high3, htf_low3, htf_time2] = request.security(syminfo.tickerid, tf, [high[1], low[1], high[3], low[3], time[2]], lookahead = barmerge.lookahead_on)
        isBullishFVG = htf_high3 < htf_low1
        isBearishFVG = htf_low3 > htf_high1

        // Bullish FVG process
        if isBullishFVG
            //find such FVG has already existed.
            bool isDuplicated=false
            for [ie, fvgE] in fvgBox
                if (fvgE.top==htf_low1) and (fvgE.bottom==htf_high3)
                    isDuplicated:=true

            if not isDuplicated
                // Add FVG into FVG's array
                FVG fvg1 = FVG.new(tf_idx, tf, tf_label, tf_num, na, true, false, false, left=bar_index - 2, top=htf_low1, right=0, bottom=htf_high3, extend = extend.none, border_style=line.style_solid, border_width=1, bgcolor=bullishFvgColor, border_color=color.new(color.green, 100), _text=tf_label, text_color=color.white, text_halign=text.align_right, text_size = label_size)
                fvgBox.push(fvg1)
                log.warning("added a BullFVG."+str.tostring(tf_idx))

                // Check if FVG to show is upper than user parameter
                if(fvgBox.size() > getTFMaxFVGs(tf_idx))
                    // Delete the FVG and its index from arrays
                    log.info("deleting BullFVG:"+str.tostring(fvgBox.size())+","+str.tostring(box.all.size()))
                    box b=fvgBox.get(0)._box
                    if not na(b)
                        box.delete(b)
                    fvgBox.remove(0)

        // Bearish FVG process
        if isBearishFVG
            //find such FVG has already existed.
            bool isDuplicated=false
            for [ie, fvgE] in fvgBox
                if (fvgE.top==htf_low3) and (fvgE.bottom==htf_high1)
                    isDuplicated:=true

            if not isDuplicated
                // Add FVG into FVG's array
                FVG fvg1 = FVG.new(tf_idx, tf, tf_label, tf_num, na, false, false, false, left=bar_index - 2, top=htf_low3, right=0, bottom=htf_high1, extend = extend.none, border_style=line.style_solid, border_width=1, bgcolor=bearishFvgColor, border_color=color.new(color.red, 100), _text=tf_label, text_color=color.white, text_halign=text.align_right, text_size = label_size)
                fvgBox.push(fvg1)
                log.warning("added a BearFVG."+str.tostring(tf_idx))

                // Check if FVG to show is upper than user parameter
                if(fvgBox.size() > getTFMaxFVGs(tf_idx))
                    // Delete the FVG and its index from arrays
                    log.info("deleting BearFVG:"+str.tostring(fvgBox.size())+","+str.tostring(box.all.size()))
                    box b=fvgBox.get(0)._box
                    if not na(b)
                        box.delete(b)
                    fvgBox.remove(0)

// Draw FVG
FVGDraw () =>
    for i1 = 0 to 7
        if (getIsTFShow(i1))
            FVG[] fvgBox = switch (i1)
                0 => fvgBox1
                1 => fvgBox2
                2 => fvgBox3
                3 => fvgBox4
                4 => fvgBox5
                5 => fvgBox6
                6 => fvgBox7
                7 => fvgBox8
            // Loop into all values of the array
            for [index, value] in fvgBox
                if (not value._isDrawn)
                    value._isDrawn:=true
                    // Processing FVG
                    value._box:=box.new(left=value.left, top=value.top, right=bar_index+value.tf_num*fvgTFDistance, bottom=value.bottom, extend = value.extend, border_style=value.border_style, border_width=value.border_width, bgcolor=value.bgcolor, border_color=value.border_color, text=value._text+(isAddBarIndex?str.tostring(bar_index):""), text_color=value.text_color, text_halign=value.text_halign, text_size = value.text_size)
                    log.warning("added a box, bar,bardist:"+str.tostring(bar_index)+","+str.tostring(value.tf_num*fvgTFDistance))

                if(value._type)
                    // Check if FVG has been totally mitigated
                    if(low <= box.get_bottom(value._box))
                        box.delete(value._box)
                        fvgBox.remove(index)
                        log.info("FVGDraw bullFVG removed.")
                    else
                        if(low < box.get_top((value._box)))
                            box.set_bgcolor(value._box, mitigatedFvgColor)

                            // Mitigated FVG Alert
                            if(not(value._isMitigated))
                                alert("FVG has been mitigated", alert.freq_once_per_bar)
                                value._isMitigated:=true

                            // Reduce FVG if needed
                            if(isMitigatedFvgToReduce)
                                box.set_text(value._box, value.tf_label+"X")
                                box.set_text_formatting(value._box, text.format_italic)
                                box.set_top(value._box, low)

                        box.set_right(value._box, bar_index+ value.tf_num*fvgTFDistance)
                // Processing bearish FVG
                else
                    // Check if FVG has been mitigated
                    if(high >= box.get_top(value._box))
                        box.delete(value._box)
                        fvgBox.remove(index)
                        log.info("FVGDraw bearFVG removed.")
                    else
                        if(high > box.get_bottom(value._box))
                            box.set_bgcolor(value._box, mitigatedFvgColor)

                            // Mitigated FVG Alert
                            if(not(value._isMitigated))
                                alert("FVG has been mitigated", alert.freq_once_per_bar)
                                value._isMitigated:=true

                            // Reduce FVG if needed
                            if(isMitigatedFvgToReduce)
                                box.set_text(value._box, value.tf_label+"X")
                                box.set_text_formatting(value._box, text.format_italic)
                                box.set_bottom(value._box, high)

                        box.set_right(value._box, bar_index+ value.tf_num*fvgTFDistance)

// @function            get line style from string
// @param style         line style
// @returns             string
getStyle(string style) =>
    switch style
        SOLID => line.style_solid
        DASHED => line.style_dashed
        DOTTED => line.style_dotted

// @function            draw MultiTimeFrame levels
// @param timeframe     base timeframe
// @param sameTimeframe true if chart timeframe is same as base timeframe
// @param style         line style
// @param levelColor    line and text color
// @returns             void
drawLevels(string timeframe, bool sameTimeframe, string style, color levelColor) =>
    [topLevel, bottomLevel, leftTime, rightTime] = request.security(syminfo.tickerid, timeframe, [high[1], low[1], time[1], time],lookahead = barmerge.lookahead_on)

    float parsedTop         = sameTimeframe ? high : topLevel
    float parsedBottom      = sameTimeframe ? low : bottomLevel    

    int parsedLeftTime      = sameTimeframe ? time : leftTime
    int parsedRightTime     = sameTimeframe ? time : rightTime

    int parsedTopTime       = time
    int parsedBottomTime    = time

    if not sameTimeframe
        int leftIndex               = times.binary_search_rightmost(parsedLeftTime)
        int rightIndex              = times.binary_search_rightmost(parsedRightTime)

        array<int> timeArray        = times.slice(leftIndex,rightIndex)
        array<float> topArray       = highs.slice(leftIndex,rightIndex)
        array<float> bottomArray    = lows.slice(leftIndex,rightIndex)

        parsedTopTime               := timeArray.size() > 0 ? timeArray.get(topArray.indexof(topArray.max())) : initialTime
        parsedBottomTime            := timeArray.size() > 0 ? timeArray.get(bottomArray.indexof(bottomArray.min())) : initialTime

    var line topLine        = line.new(na, na, na, na, xloc = xloc.bar_time, color = levelColor, style = getStyle(style))
    var line bottomLine     = line.new(na, na, na, na, xloc = xloc.bar_time, color = levelColor, style = getStyle(style))
    var label topLabel      = label.new(na, na, xloc = xloc.bar_time, text = str.format('P{0}H',timeframe), color=color(na), textcolor = levelColor, size = size.small, style = label.style_label_left)
    var label bottomLabel   = label.new(na, na, xloc = xloc.bar_time, text = str.format('P{0}L',timeframe), color=color(na), textcolor = levelColor, size = size.small, style = label.style_label_left)

    topLine.set_first_point(    chart.point.new(parsedTopTime,na,parsedTop))
    topLine.set_second_point(   chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedTop))   
    topLabel.set_point(         chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedTop))

    bottomLine.set_first_point( chart.point.new(parsedBottomTime,na,parsedBottom))    
    bottomLine.set_second_point(chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedBottom))
    bottomLabel.set_point(      chart.point.new(last_bar_time + 20 * (time-time[1]),na,parsedBottom))

// @function            true if chart timeframe is higher than provided timeframe
// @param timeframe     timeframe to check
// @returns             bool
higherTimeframe(string timeframe) => timeframe.in_seconds() > timeframe.in_seconds(timeframe)

// @function            update trailing swing points
// @returns             int
updateTrailingExtremes() =>
    trailing.top            := math.max(high,trailing.top)
    trailing.lastTopTime    := trailing.top == high ? time : trailing.lastTopTime
    trailing.bottom         := math.min(low,trailing.bottom)
    trailing.lastBottomTime := trailing.bottom == low ? time : trailing.lastBottomTime

// @function            draw trailing swing points
// @returns             void
drawHighLowSwings() =>
    var line topLine        = line.new(na, na, na, na, color = swingBearishColor, xloc = xloc.bar_time)
    var line bottomLine     = line.new(na, na, na, na, color = swingBullishColor, xloc = xloc.bar_time)
    var label topLabel      = label.new(na, na, color=color(na), textcolor = swingBearishColor, xloc = xloc.bar_time, style = label.style_label_down, size = size.tiny)
    var label bottomLabel   = label.new(na, na, color=color(na), textcolor = swingBullishColor, xloc = xloc.bar_time, style = label.style_label_up, size = size.tiny)

    rightTimeBar            = last_bar_time + 20 * (time - time[1])

    topLine.set_first_point(    chart.point.new(trailing.lastTopTime, na, trailing.top))
    topLine.set_second_point(   chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_point(         chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_text(          swingTrend.bias == BEARISH ? 'Strong High' : 'Weak High')

    bottomLine.set_first_point( chart.point.new(trailing.lastBottomTime, na, trailing.bottom))
    bottomLine.set_second_point(chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_point(      chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_text(       swingTrend.bias == BULLISH ? 'Strong Low' : 'Weak Low')

// @function            draw a zone with a label and a box
// @param labelLevel    price level for label
// @param labelIndex    bar index for label
// @param top           top price level for box
// @param bottom        bottom price level for box
// @param tag           text to display
// @param zoneColor     base color
// @param style         label style
// @returns             void
drawZone(float labelLevel, int labelIndex, float top, float bottom, string tag, color zoneColor, string style) =>
    var label l_abel    = label.new(na,na,text = tag, color=color(na),textcolor = zoneColor, style = style, size = size.small)
    var box b_ox        = box.new(na,na,na,na,bgcolor = color.new(zoneColor,80),border_color = color(na), xloc = xloc.bar_time)

    b_ox.set_top_left_point(    chart.point.new(trailing.barTime,na,top))
    b_ox.set_bottom_right_point(chart.point.new(last_bar_time,na,bottom))

    l_abel.set_point(           chart.point.new(na,labelIndex,labelLevel))

// @function            draw premium/discount zones
// @returns             void
getIsTFShow(int tf_idx) =>
    k = switch tf_idx
        0 => tf1_show
        1 => tf2_show
        2 => tf3_show
        3 => tf4_show
        4 => tf5_show
        5 => tf6_show
        6 => tf7_show
        7 => tf8_show

getTFLabel(int tf_idx) =>
    k = switch tf_idx
        0 => tf1_label
        1 => tf2_label
        2 => tf3_label
        3 => tf4_label
        4 => tf5_label
        5 => tf6_label
        6 => tf7_label
        7 => tf8_label

getTFMaxFVGs(int tf_idx) =>
    k = switch tf_idx
        0 => math.min(tf1_fvg_limit, fvgHistoryNbr)
        1 => math.min(tf2_fvg_limit, fvgHistoryNbr)
        2 => math.min(tf3_fvg_limit, fvgHistoryNbr)
        3 => math.min(tf4_fvg_limit, fvgHistoryNbr)
        4 => math.min(tf5_fvg_limit, fvgHistoryNbr)
        5 => math.min(tf6_fvg_limit, fvgHistoryNbr)
        6 => math.min(tf7_fvg_limit, fvgHistoryNbr)
        7 => math.min(tf8_fvg_limit, fvgHistoryNbr)
drawPremiumDiscountZones() =>
    drawZone(trailing.top, math.round(0.5*(trailing.barIndex + last_bar_index)), trailing.top, 0.95*trailing.top + 0.05*trailing.bottom, 'Premium', premiumZoneColor, label.style_label_down)

    equilibriumLevel = math.avg(trailing.top, trailing.bottom)
    drawZone(equilibriumLevel, last_bar_index, 0.525*trailing.top + 0.475*trailing.bottom, 0.525*trailing.bottom + 0.475*trailing.top, 'Equilibrium', equilibriumZoneColorInput, label.style_label_left)

    drawZone(trailing.bottom, math.round(0.5*(trailing.barIndex + last_bar_index)), 0.95*trailing.bottom + 0.05*trailing.top, trailing.bottom, 'Discount', discountZoneColor, label.style_label_up)

// @function            detect and mark fractal highs
// @returns             bool - true if new fractal high detected
detectFractalHigh() =>
    bool newFractal = false
    
    if bar_index >= fractalLookbackInput
        isFractalHigh = true
        
        // Check if current bar's high is higher than lookback bars
        for i = 1 to fractalLookbackInput
            if high[i] >= high[0]
                isFractalHigh := false
                break
        
        // Check if next bar failed to break the high (confirmation)
        if isFractalHigh and bar_index > fractalLookbackInput
            wasHighBroken = false
            for i = 1 to fractalLookbackInput
                if high[i] > high[fractalLookbackInput]
                    wasHighBroken := true
                    break
            
            if not wasHighBroken
                newFractal := true
                fractalPoint fh = fractalPoint.new(high[fractalLookbackInput], time[fractalLookbackInput], bar_index - fractalLookbackInput, false, true)
                fractalHighs.unshift(fh)
                
                // Limit array size
                if fractalHighs.size() > 50
                    fractalHighs.pop()
    
    newFractal

// @function            detect and mark fractal lows
// @returns             bool - true if new fractal low detected
detectFractalLow() =>
    bool newFractal = false
    
    if bar_index >= fractalLookbackInput
        isFractalLow = true
        
        // Check if current bar's low is lower than lookback bars
        for i = 1 to fractalLookbackInput
            if low[i] <= low[0]
                isFractalLow := false
                break
        
        // Check if next bar failed to break the low (confirmation)
        if isFractalLow and bar_index > fractalLookbackInput
            wasLowBroken = false
            for i = 1 to fractalLookbackInput
                if low[i] < low[fractalLookbackInput]
                    wasLowBroken := true
                    break
            
            if not wasLowBroken
                newFractal := true
                fractalPoint fl = fractalPoint.new(low[fractalLookbackInput], time[fractalLookbackInput], bar_index - fractalLookbackInput, true, true)
                fractalLows.unshift(fl)
                
                // Limit array size
                if fractalLows.size() > 50
                    fractalLows.pop()
    
    newFractal

// @function            determine fractal structure (bullish/bearish)
// @returns             int - updated structure bias
updateFractalStructure() =>
    int newBias = fractalStructureBias
    
    if fractalHighs.size() > 0 and fractalLows.size() > 0
        lastFractalHigh = fractalHighs.get(0)
        lastFractalLow = fractalLows.get(0)
        
        // Check if price broke recent fractal high (bullish)
        if high > lastFractalHigh.price
            newBias := BULLISH
        
        // Check if price broke recent fractal low (bearish)
        if low < lastFractalLow.price
            newBias := BEARISH
    
    newBias

// @function            draw fractal structure lines and labels
// @returns             void
drawFractalStructure() =>
    if showFractalsInput
        // Draw structure lines between fractals
        if fractalHighs.size() >= 2 and fractalLows.size() >= 2
            lastHigh = fractalHighs.get(0)
            lastLow = fractalLows.get(0)
            
            // Determine which came first and draw line
            if lastHigh.barIndex > lastLow.barIndex
                // Last fractal was a high, connect to previous low
                if fractalLows.size() > 1
                    prevLow = fractalLows.get(1)
                    structureColor = lastHigh.price > fractalHighs.get(1).price ? fractalBullishColorInput : fractalBearishColorInput
                    line.new(chart.point.new(prevLow.barTime, na, prevLow.price), 
                             chart.point.new(lastHigh.barTime, na, lastHigh.price), 
                             xloc = xloc.bar_time, 
                             color = structureColor, 
                             width = fractalLineWidthInput)
            else
                // Last fractal was a low, connect to previous high
                if fractalHighs.size() > 1
                    prevHigh = fractalHighs.get(1)
                    structureColor = lastLow.price < fractalLows.get(1).price ? fractalBearishColorInput : fractalBullishColorInput
                    line.new(chart.point.new(prevHigh.barTime, na, prevHigh.price), 
                             chart.point.new(lastLow.barTime, na, lastLow.price), 
                             xloc = xloc.bar_time, 
                             color = structureColor, 
                             width = fractalLineWidthInput)

//---------------------------------------------------------------------------------------------------------------------}
//MUTABLE VARIABLES & EXECUTION
//---------------------------------------------------------------------------------------------------------------------{
[lastP, startPrice, height] = update()

if is_enable_autofib_section
    lineId0 = processLevel(show_neg_0_65, value_neg_0_65, color_neg_0_65, line(na))
    lineId1 = processLevel(show_neg_0_618, value_neg_0_618, color_neg_0_618, lineId0)
    lineId2 = processLevel(show_neg_0_382, value_neg_0_382, color_neg_0_382, lineId1)
    lineId3 = processLevel(show_neg_0_236, value_neg_0_236, color_neg_0_236, lineId2)
    lineId4 = processLevel(show_0, value_0, color_0, lineId3)
    lineId5 = processLevel(show_0_236, value_0_236, color_0_236, lineId4)
    lineId6 = processLevel(show_0_382, value_0_382, color_0_382, lineId5)
    lineId7 = processLevel(show_0_5, value_0_5, color_0_5, lineId6)
    lineId8 = processLevel(show_0_618, value_0_618, color_0_618, lineId7)
    lineId9 = processLevel(show_0_65, value_0_65, color_0_65, lineId8)
    lineId10 = processLevel(show_0_786, value_0_786, color_0_786, lineId9)
    lineId11 = processLevel(show_1, value_1, color_1, lineId10)
    lineId12 = processLevel(show_1_272, value_1_272, color_1_272, lineId11)
    lineId13 = processLevel(show_1_414, value_1_414, color_1_414, lineId12)
    lineId14 = processLevel(show_1_618, value_1_618, color_1_618, lineId13)
    lineId15 = processLevel(show_1_65, value_1_65, color_1_65, lineId14)
    lineId16 = processLevel(show_2_618, value_2_618, color_2_618, lineId15)
    lineId17 = processLevel(show_2_65, value_2_65, color_2_65, lineId16)
    lineId18 = processLevel(show_3_618, value_3_618, color_3_618, lineId17)
    lineId19 = processLevel(show_3_65, value_3_65, color_3_65, lineId18)
    lineId20 = processLevel(show_4_236, value_4_236, color_4_236, lineId19)
    lineId21 = processLevel(show_4_618, value_4_618, color_4_618, lineId20)
// Candle coloring
parsedOpen  = showTrendInput ? open : na
candleColor = internalTrend.bias == BULLISH ? swingBullishColor : swingBearishColor
plotcandle(parsedOpen,high,low,close,color = candleColor, wickcolor = candleColor, bordercolor = candleColor)

if showHighLowSwingsInput or showPremiumDiscountZonesInput
    updateTrailingExtremes()

    if showHighLowSwingsInput
        drawHighLowSwings()

    if showPremiumDiscountZonesInput
        drawPremiumDiscountZones()

// Fractal Structure Tracker
newFractalHigh = detectFractalHigh()
newFractalLow = detectFractalLow()

prevFractalStructureBias := fractalStructureBias
fractalStructureBias := updateFractalStructure()

drawFractalStructure()

// Plot fractal shapes and labels
plotshape(showFractalsInput and showBearishFractalsInput and newFractalHigh,
          title='Fractal High', location=location.abovebar, style=shape.triangledown,
          color=fractalBearishColorInput, size=size.small,
          offset=-fractalLookbackInput)

if showFractalsInput and showBearishFractalsInput and newFractalHigh and showFractalLabelsInput
    fh = fractalHighs.get(0)
    label.new(chart.point.new(fh.barTime, na, fh.price),
              'FH', xloc = xloc.bar_time, color = color(na),
              textcolor = fractalBearishColorInput,
              style = label.style_label_down, size = size.tiny)

plotshape(showFractalsInput and showBullishFractalsInput and newFractalLow,
          title='Fractal Low', location=location.belowbar, style=shape.triangleup,
          color=fractalBullishColorInput, size=size.small,
          offset=-fractalLookbackInput)

if showFractalsInput and showBullishFractalsInput and newFractalLow and showFractalLabelsInput
    fl = fractalLows.get(0)
    label.new(chart.point.new(fl.barTime, na, fl.price),
              'FL', xloc = xloc.bar_time, color = color(na),
              textcolor = fractalBullishColorInput,
              style = label.style_label_up, size = size.tiny)

// Background highlight for structure change
structureChanged = prevFractalStructureBias != fractalStructureBias and fractalStructureBias != 0
bgcolor(highlightStructureChangeInput and structureChanged ? color.new(fractalStructureBias == BULLISH ? fractalBullishColorInput : fractalBearishColorInput, 90) : na, title = 'Structure Change Highlight')

getCurrentStructure(swingsLengthInput,false)
getCurrentStructure(5,false,true)

if showEqualHighsLowsInput
    getCurrentStructure(equalHighsLowsLengthInput,true)

if showInternalsInput or showInternalOrderBlocksInput or showTrendInput
    displayStructure(true)

if showStructureInput or showSwingOrderBlocksInput or showHighLowSwingsInput
    displayStructure()

if showInternalOrderBlocksInput
    deleteOrderBlocks(true)

if showSwingOrderBlocksInput
    deleteOrderBlocks()

if is_enable_fvg_section
    int tf_num=0
    if tf8_show
        findFVGs(7, tf8, tf8_label, tf_num, fvgBox8)
        tf_num:=tf_num+1
    if tf7_show
        findFVGs(6, tf7, tf7_label, tf_num, fvgBox7)
        tf_num:=tf_num+1
    if tf6_show
        findFVGs(5, tf6, tf6_label, tf_num, fvgBox6)
        tf_num:=tf_num+1
    if tf5_show
        findFVGs(4, tf5, tf5_label, tf_num, fvgBox5)
        tf_num:=tf_num+1
    if tf4_show
        findFVGs(3, tf4, tf4_label, tf_num, fvgBox4)
        tf_num:=tf_num+1
    if tf3_show
        findFVGs(2, tf3, tf3_label, tf_num, fvgBox3)
        tf_num:=tf_num+1
    if tf2_show
        findFVGs(1, tf2, tf2_label, tf_num, fvgBox2)
        tf_num:=tf_num+1
    if tf1_show
        findFVGs(0, tf1, tf1_label, tf_num, fvgBox1)
        tf_num:=tf_num+1
    FVGDraw()

if barstate.islastconfirmedhistory or barstate.islast
    if showInternalOrderBlocksInput        
        drawOrderBlocks(true)
        
    if showSwingOrderBlocksInput        
        drawOrderBlocks()

lastBarIndex    := currentBarIndex
currentBarIndex := bar_index
newBar          = currentBarIndex != lastBarIndex

if barstate.islastconfirmedhistory or (barstate.isrealtime and newBar)
    if showDailyLevelsInput and not higherTimeframe('D')
        drawLevels('D',timeframe.isdaily,dailyLevelsStyleInput,dailyLevelsColorInput)

    if showWeeklyLevelsInput and not higherTimeframe('W')
        drawLevels('W',timeframe.isweekly,weeklyLevelsStyleInput,weeklyLevelsColorInput)

    if showMonthlyLevelsInput and not higherTimeframe('M')
        drawLevels('M',timeframe.ismonthly,monthlyLevelsStyleInput,monthlyLevelsColorInput)

//---------------------------------------------------------------------------------------------------------------------}
// TABLE
//---------------------------------------------------------------------------------------------------------------------{
var table_ = table.new(position.bottom_right, 6, 2, bgcolor = color.new(color.gray, 80), border_width = 1)

if barstate.islast
    table.cell(table_, 0, 0, "Moving Average", text_color = color.white)
    table.cell(table_, 1, 0, "Structure", text_color = color.white)
    table.cell(table_, 2, 0, "news", text_color = color.white)
    table.cell(table_, 3, 0, "Entry", text_color = color.white)
    table.cell(table_, 4, 0, "SL", text_color = color.white)
    table.cell(table_, 5, 0, "TP", text_color = color.white)
    
    table.cell(table_, 0, 1, "", text_color = color.white)
    table.cell(table_, 1, 1, "", text_color = color.white)
    table.cell(table_, 2, 1, "", text_color = color.white)
    table.cell(table_, 3, 1, "", text_color = color.white)
    table.cell(table_, 4, 1, "", text_color = color.white)
    table.cell(table_, 5, 1, "", text_color = color.white)

//=========== MA PART============//


//************  Globals
// =====================================================================
// 1. COLOR PALETTES PER TEMPLATE (8 templates × 10 colors)
// =====================================================================
colorSet tplColors_1 = colorSet.new(#1565C0, #FF3232, #800080, color.white, color.yellow, color.green, #e17d9e, #40E0D0, #82FFC9, #00EEFF)  // EMA 20-50-100-200
colorSet tplColors_2 = colorSet.new(#1E88E5, #E53935, #8E24AA, #B0BEC5, #FFEB3B, #43A047, #F06292, #26C6DA, #81C784, #00E5FF)  // SMA 5-10-20-50-200
colorSet tplColors_3 = colorSet.new(#1976D2, #D32F2F, #7B1FA2, #F5F5F5, #FFEE58, #388E3C, #EC407A, #00BCD4, #66BB6A, #00B8D4)  // Fibonacci
colorSet tplColors_4 = colorSet.new(#1565C0, #E53935, #6A1B9A, color.white, #FFB300, #43A047, #F06292, #00ACC1, #81C784, #00B8D4)  // EMA/SMA Mix
colorSet tplColors_5 = colorSet.new(#1E88E5, #D32F2F, #8E24AA, #B0BEC5, #FFEB3B, #43A047, #F06292, #26C6DA, #81C784, #00E5FF)  // Golden EMA
colorSet tplColors_6 = colorSet.new(#1976D2, #C62828, #7B1FA2, #F5F5F5, #FFEE58, #388E3C, #EC407A, #00BCD4, #66BB6A, #00B8D4)  // Golden SMA
colorSet tplColors_7 = colorSet.new(#0288D1, #D50000, #9C27B0, #FAFAFA, #FFF176, #2E7D32, #E91E63, #00BFA5, #69F0AE, #00E676)  // HMA 21-34
colorSet tplColors_8 = colorSet.new(#0D47A1, #B71C1C, #4A148C, #EEEEEE, #FFCA28, #1B5E20, #AD1457, #006064, #1DE9B6, #00E5FF)  // The Crypto School Trading
//colorSet tplColors_default = colorSet.new(#0D47A1, #B71C1C, #4A148C, #EEEEEE, #FFCA28, #1B5E20, #AD1457, #006064, #1DE9B6, #00E5FF)  // Default

// =====================================================================
// 2. TEMPLATE MA SETTINGS (8 × 10) – Arrays
// =====================================================================
//********** Templates: settings

//----- EMA 20-50-100-200
string tpl_type1_1_MaType = 'EMA'
int tpl_type1_1_MaLength = 20
string tpl_type1_2_MaType = 'EMA'
int tpl_type1_2_MaLength = 50
string tpl_type1_3_MaType = 'EMA'
int tpl_type1_3_MaLength = 100
string tpl_type1_4_MaType = 'EMA'
int tpl_type1_4_MaLength = 200
string tpl_type1_5_MaType = 'DISABLED'
int tpl_type1_5_MaLength = 1
string tpl_type1_6_MaType = 'DISABLED'
int tpl_type1_6_MaLength = 1
string tpl_type1_7_MaType = 'DISABLED'
int tpl_type1_7_MaLength = 1
string tpl_type1_8_MaType = 'DISABLED'
int tpl_type1_8_MaLength = 1
string tpl_type1_9_MaType = 'DISABLED'
int tpl_type1_9_MaLength = 1
string tpl_type1_10_MaType = 'DISABLED'
int tpl_type1_10_MaLength = 1


//----- SMA 5-10-20-50-200
string tpl_type2_1_MaType = 'SMA'
int tpl_type2_1_MaLength = 5
string tpl_type2_2_MaType = 'SMA'
int tpl_type2_2_MaLength = 10
string tpl_type2_3_MaType = 'SMA'
int tpl_type2_3_MaLength = 20
string tpl_type2_4_MaType = 'SMA'
int tpl_type2_4_MaLength = 50
string tpl_type2_5_MaType = 'SMA'
int tpl_type2_5_MaLength = 200
string tpl_type2_6_MaType = 'DISABLED'
int tpl_type2_6_MaLength = 1
string tpl_type2_7_MaType = 'DISABLED'
int tpl_type2_7_MaLength = 1
string tpl_type2_8_MaType = 'DISABLED'
int tpl_type2_8_MaLength = 1
string tpl_type2_9_MaType = 'DISABLED'
int tpl_type2_9_MaLength = 1
string tpl_type2_10_MaType = 'DISABLED'
int tpl_type2_10_MaLength = 1


//----- EMA Fibonacci 21-55-89-144-233
// Fibonacci Sequences
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610
string tpl_type3_1_MaType = 'EMA'
int tpl_type3_1_MaLength = 21
string tpl_type3_2_MaType = 'EMA'
int tpl_type3_2_MaLength = 55
string tpl_type3_3_MaType = 'EMA'
int tpl_type3_3_MaLength = 89
string tpl_type3_4_MaType = 'EMA'
int tpl_type3_4_MaLength = 144
string tpl_type3_5_MaType = 'EMA'
int tpl_type3_5_MaLength = 233
string tpl_type3_6_MaType = 'DISABLED'
int tpl_type3_6_MaLength = 1
string tpl_type3_7_MaType = 'DISABLED'
int tpl_type3_7_MaLength = 1
string tpl_type3_8_MaType = 'DISABLED'
int tpl_type3_8_MaLength = 1
string tpl_type3_9_MaType = 'DISABLED'
int tpl_type3_9_MaLength = 1
string tpl_type3_10_MaType = 'DISABLED'
int tpl_type3_10_MaLength = 1


//----- EMA 20-50-100-200 / SMA 100-200
string tpl_type4_1_MaType = 'EMA'
int tpl_type4_1_MaLength = 20
string tpl_type4_2_MaType = 'EMA'
int tpl_type4_2_MaLength = 50
string tpl_type4_3_MaType = 'EMA'
int tpl_type4_3_MaLength = 100
string tpl_type4_4_MaType = 'EMA'
int tpl_type4_4_MaLength = 200
string tpl_type4_5_MaType = 'SMA'
int tpl_type4_5_MaLength = 100
string tpl_type4_6_MaType = 'SMA'
int tpl_type4_6_MaLength = 100
string tpl_type4_7_MaType = 'DISABLED'
int tpl_type4_7_MaLength = 1
string tpl_type4_8_MaType = 'DISABLED'
int tpl_type4_8_MaLength = 1
string tpl_type4_9_MaType = 'DISABLED'
int tpl_type4_9_MaLength = 1
string tpl_type4_10_MaType = 'DISABLED'
int tpl_type4_10_MaLength = 1


//----- EMA Golden/Death Cross 50-200
string tpl_type5_1_MaType = 'EMA'
int tpl_type5_1_MaLength = 50
string tpl_type5_2_MaType = 'EMA'
int tpl_type5_2_MaLength = 200
string tpl_type5_3_MaType = 'DISABLED'
int tpl_type5_3_MaLength = 1
string tpl_type5_4_MaType = 'DISABLED'
int tpl_type5_4_MaLength = 1
string tpl_type5_5_MaType = 'DISABLED'
int tpl_type5_5_MaLength = 1
string tpl_type5_6_MaType = 'DISABLED'
int tpl_type5_6_MaLength = 1
string tpl_type5_7_MaType = 'DISABLED'
int tpl_type5_7_MaLength = 1
string tpl_type5_8_MaType = 'DISABLED'
int tpl_type5_8_MaLength = 1
string tpl_type5_9_MaType = 'DISABLED'
int tpl_type5_9_MaLength = 1
string tpl_type5_10_MaType = 'DISABLED'
int tpl_type5_10_MaLength = 1


//----- SMA Golden/Death Cross 50-200
string tpl_type6_1_MaType = 'SMA'
int tpl_type6_1_MaLength = 50
string tpl_type6_2_MaType = 'SMA'
int tpl_type6_2_MaLength = 200
string tpl_type6_3_MaType = 'DISABLED'
int tpl_type6_3_MaLength = 1
string tpl_type6_4_MaType = 'DISABLED'
int tpl_type6_4_MaLength = 1
string tpl_type6_5_MaType = 'DISABLED'
int tpl_type6_5_MaLength = 1
string tpl_type6_6_MaType = 'DISABLED'
int tpl_type6_6_MaLength = 1
string tpl_type6_7_MaType = 'DISABLED'
int tpl_type6_7_MaLength = 1
string tpl_type6_8_MaType = 'DISABLED'
int tpl_type6_8_MaLength = 1
string tpl_type6_9_MaType = 'DISABLED'
int tpl_type6_9_MaLength = 1
string tpl_type6_10_MaType = 'DISABLED'
int tpl_type6_10_MaLength = 1


//----- HMA Cross 21-34
string tpl_type7_1_MaType = 'HMA'
int tpl_type7_1_MaLength = 21
string tpl_type7_2_MaType = 'HMA'
int tpl_type7_2_MaLength = 34
string tpl_type7_3_MaType = 'DISABLED'
int tpl_type7_3_MaLength = 1
string tpl_type7_4_MaType = 'DISABLED'
int tpl_type7_4_MaLength = 1
string tpl_type7_5_MaType = 'DISABLED'
int tpl_type7_5_MaLength = 1
string tpl_type7_6_MaType = 'DISABLED'
int tpl_type7_6_MaLength = 1
string tpl_type7_7_MaType = 'DISABLED'
int tpl_type7_7_MaLength = 1
string tpl_type7_8_MaType = 'DISABLED'
int tpl_type7_8_MaLength = 1
string tpl_type7_9_MaType = 'DISABLED'
int tpl_type7_9_MaLength = 1
string tpl_type7_10_MaType = 'DISABLED'
int tpl_type7_10_MaLength = 1


//----- The Crypto School Long Short Trading
string tpl_type8_1_MaType = 'SMA'
int tpl_type8_1_MaLength = 21
string tpl_type8_2_MaType = 'SMA'
int tpl_type8_2_MaLength = 50
string tpl_type8_3_MaType = 'SMA'
int tpl_type8_3_MaLength = 200
string tpl_type8_4_MaType = 'EMA'
int tpl_type8_4_MaLength = 50
string tpl_type8_5_MaType = 'SMMA (RMA)'
int tpl_type8_5_MaLength = 7
string tpl_type8_6_MaType = 'SMMA (RMA)'
int tpl_type8_6_MaLength = 9
string tpl_type8_7_MaType = 'SMMA (RMA)'
int tpl_type8_7_MaLength = 99
string tpl_type8_8_MaType = 'SMA'
int tpl_type8_8_MaLength = 9
string tpl_type8_9_MaType = 'DISABLED'
int tpl_type8_9_MaLength = 1
string tpl_type8_10_MaType = 'DISABLED'
int tpl_type8_10_MaLength = 1


// =====================================================================
// 3. INPUTS
// =====================================================================

//************  set final settings
[valFinal_1_MaType, valFinal_1_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_1_MaType, tpl_type2_1_MaType, tpl_type3_1_MaType, tpl_type4_1_MaType, tpl_type5_1_MaType, tpl_type6_1_MaType, tpl_type7_1_MaType, tpl_type8_1_MaType, input_1_MaType, tpl_type1_1_MaLength, tpl_type2_1_MaLength, tpl_type3_1_MaLength, tpl_type4_1_MaLength, tpl_type5_1_MaLength, tpl_type6_1_MaLength, tpl_type7_1_MaLength, tpl_type8_1_MaLength, input_1_MaLength)
[valFinal_2_MaType, valFinal_2_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_2_MaType, tpl_type2_2_MaType, tpl_type3_2_MaType, tpl_type4_2_MaType, tpl_type5_2_MaType, tpl_type6_2_MaType, tpl_type7_2_MaType, tpl_type8_2_MaType, input_2_MaType, tpl_type1_2_MaLength, tpl_type2_2_MaLength, tpl_type3_2_MaLength, tpl_type4_2_MaLength, tpl_type5_2_MaLength, tpl_type6_2_MaLength, tpl_type7_2_MaLength, tpl_type8_2_MaLength, input_2_MaLength)
[valFinal_3_MaType, valFinal_3_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_3_MaType, tpl_type2_3_MaType, tpl_type3_3_MaType, tpl_type4_3_MaType, tpl_type5_3_MaType, tpl_type6_3_MaType, tpl_type7_3_MaType, tpl_type8_3_MaType, input_3_MaType, tpl_type1_3_MaLength, tpl_type2_3_MaLength, tpl_type3_3_MaLength, tpl_type4_3_MaLength, tpl_type5_3_MaLength, tpl_type6_3_MaLength, tpl_type7_3_MaLength, tpl_type8_3_MaLength, input_3_MaLength)
[valFinal_4_MaType, valFinal_4_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_4_MaType, tpl_type2_4_MaType, tpl_type3_4_MaType, tpl_type4_4_MaType, tpl_type5_4_MaType, tpl_type6_4_MaType, tpl_type7_4_MaType, tpl_type8_4_MaType, input_4_MaType, tpl_type1_4_MaLength, tpl_type2_4_MaLength, tpl_type3_4_MaLength, tpl_type4_4_MaLength, tpl_type5_4_MaLength, tpl_type6_4_MaLength, tpl_type7_4_MaLength, tpl_type8_4_MaLength, input_4_MaLength)
[valFinal_5_MaType, valFinal_5_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_5_MaType, tpl_type2_5_MaType, tpl_type3_5_MaType, tpl_type4_5_MaType, tpl_type5_5_MaType, tpl_type6_5_MaType, tpl_type7_5_MaType, tpl_type8_5_MaType, input_5_MaType, tpl_type1_5_MaLength, tpl_type2_5_MaLength, tpl_type3_5_MaLength, tpl_type4_5_MaLength, tpl_type5_5_MaLength, tpl_type6_5_MaLength, tpl_type7_5_MaLength, tpl_type8_5_MaLength, input_5_MaLength)
[valFinal_6_MaType, valFinal_6_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_6_MaType, tpl_type2_6_MaType, tpl_type3_6_MaType, tpl_type4_6_MaType, tpl_type5_6_MaType, tpl_type6_6_MaType, tpl_type7_6_MaType, tpl_type8_6_MaType, input_6_MaType, tpl_type1_6_MaLength, tpl_type2_6_MaLength, tpl_type3_6_MaLength, tpl_type4_6_MaLength, tpl_type5_6_MaLength, tpl_type6_6_MaLength, tpl_type7_6_MaLength, tpl_type8_6_MaLength, input_6_MaLength)
[valFinal_7_MaType, valFinal_7_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_7_MaType, tpl_type2_7_MaType, tpl_type3_7_MaType, tpl_type4_7_MaType, tpl_type5_7_MaType, tpl_type6_7_MaType, tpl_type7_7_MaType, tpl_type8_7_MaType, input_7_MaType, tpl_type1_7_MaLength, tpl_type2_7_MaLength, tpl_type3_7_MaLength, tpl_type4_7_MaLength, tpl_type5_7_MaLength, tpl_type6_7_MaLength, tpl_type7_7_MaLength, tpl_type8_7_MaLength, input_7_MaLength)
[valFinal_8_MaType, valFinal_8_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_8_MaType, tpl_type2_8_MaType, tpl_type3_8_MaType, tpl_type4_8_MaType, tpl_type5_8_MaType, tpl_type6_8_MaType, tpl_type7_8_MaType, tpl_type8_8_MaType, input_8_MaType, tpl_type1_8_MaLength, tpl_type2_8_MaLength, tpl_type3_8_MaLength, tpl_type4_8_MaLength, tpl_type5_8_MaLength, tpl_type6_8_MaLength, tpl_type7_8_MaLength, tpl_type8_8_MaLength, input_8_MaLength)
[valFinal_9_MaType, valFinal_9_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_9_MaType, tpl_type2_9_MaType, tpl_type3_9_MaType, tpl_type4_9_MaType, tpl_type5_9_MaType, tpl_type6_9_MaType, tpl_type7_9_MaType, tpl_type8_9_MaType, input_9_MaType, tpl_type1_9_MaLength, tpl_type2_9_MaLength, tpl_type3_9_MaLength, tpl_type4_9_MaLength, tpl_type5_9_MaLength, tpl_type6_9_MaLength, tpl_type7_9_MaLength, tpl_type8_9_MaLength, input_9_MaLength)
[valFinal_10_MaType, valFinal_10_MaLength] = _funcSetFinalVal(inputTemplate, tpl_type1_10_MaType, tpl_type2_10_MaType, tpl_type3_10_MaType, tpl_type4_10_MaType, tpl_type5_10_MaType, tpl_type6_10_MaType, tpl_type7_10_MaType, tpl_type8_10_MaType, input_10_MaType, tpl_type1_10_MaLength, tpl_type2_10_MaLength, tpl_type3_10_MaLength, tpl_type4_10_MaLength, tpl_type5_10_MaLength, tpl_type6_10_MaLength, tpl_type7_10_MaLength, tpl_type8_10_MaLength, input_10_MaLength)

//************ Plotting
if inputTemplate!="DISABLED"
    input_1_Timeframe:=input_template_Timeframe
    input_2_Timeframe:=input_template_Timeframe
    input_3_Timeframe:=input_template_Timeframe
    input_4_Timeframe:=input_template_Timeframe
    input_5_Timeframe:=input_template_Timeframe
    input_6_Timeframe:=input_template_Timeframe
    input_7_Timeframe:=input_template_Timeframe
    input_8_Timeframe:=input_template_Timeframe
    input_9_Timeframe:=input_template_Timeframe
    input_10_Timeframe:=input_template_Timeframe

ma1= input_1_showMA ? _funcCalcMA(valFinal_1_MaType, input_1_Source, input_1_Timeframe, valFinal_1_MaLength): na
ma2= input_2_showMA ? _funcCalcMA(valFinal_2_MaType, input_2_Source, input_2_Timeframe, valFinal_2_MaLength): na
ma3= input_3_showMA ? _funcCalcMA(valFinal_3_MaType, input_3_Source, input_3_Timeframe, valFinal_3_MaLength): na
ma4= input_4_showMA ? _funcCalcMA(valFinal_4_MaType, input_4_Source, input_4_Timeframe, valFinal_4_MaLength): na
ma5= input_5_showMA ? _funcCalcMA(valFinal_5_MaType, input_5_Source, input_5_Timeframe, valFinal_5_MaLength): na
ma6= input_6_showMA ? _funcCalcMA(valFinal_6_MaType, input_6_Source, input_6_Timeframe, valFinal_6_MaLength): na
ma7= input_7_showMA ? _funcCalcMA(valFinal_7_MaType, input_7_Source, input_7_Timeframe, valFinal_7_MaLength): na
ma8= input_8_showMA ? _funcCalcMA(valFinal_8_MaType, input_8_Source, input_8_Timeframe, valFinal_8_MaLength): na
ma9= input_9_showMA ? _funcCalcMA(valFinal_9_MaType, input_9_Source, input_9_Timeframe, valFinal_9_MaLength): na
ma10= input_10_showMA ? _funcCalcMA(valFinal_10_MaType, input_10_Source, input_10_Timeframe, valFinal_10_MaLength): na

//ma1= _funcCalcMA(valFinal_1_MaType, input_1_Source, input_1_Timeframe, valFinal_1_MaLength)
//ma2= _funcCalcMA(valFinal_2_MaType, input_2_Source, input_2_Timeframe, valFinal_2_MaLength)
//ma3= _funcCalcMA(valFinal_3_MaType, input_3_Source, input_3_Timeframe, valFinal_3_MaLength)
//ma4= _funcCalcMA(valFinal_4_MaType, input_4_Source, input_4_Timeframe, valFinal_4_MaLength)
//ma5= _funcCalcMA(valFinal_5_MaType, input_5_Source, input_5_Timeframe, valFinal_5_MaLength)
//ma6= _funcCalcMA(valFinal_6_MaType, input_6_Source, input_6_Timeframe, valFinal_6_MaLength)
//ma7= _funcCalcMA(valFinal_7_MaType, input_7_Source, input_7_Timeframe, valFinal_7_MaLength)
//ma8= _funcCalcMA(valFinal_8_MaType, input_8_Source, input_8_Timeframe, valFinal_8_MaLength)
//ma9= _funcCalcMA(valFinal_9_MaType, input_9_Source, input_9_Timeframe, valFinal_9_MaLength)
//ma10= _funcCalcMA(valFinal_10_MaType, input_10_Source, input_10_Timeframe, valFinal_10_MaLength)


plot(ma1,  title='MA 1',  color=color.new(input_1_color, input_1_LineTransparency), linewidth=input_1_LineWidth, style=_funcLineStyle(input_1_LineStyle))
plot(ma2,  title='MA 2',  color=color.new(input_2_color, input_2_LineTransparency), linewidth=input_2_LineWidth, style=_funcLineStyle(input_2_LineStyle))
plot(ma3,  title='MA 3',  color=color.new(input_3_color, input_3_LineTransparency), linewidth=input_3_LineWidth, style=_funcLineStyle(input_3_LineStyle))
plot(ma4,  title='MA 4',  color=color.new(input_4_color, input_4_LineTransparency), linewidth=input_4_LineWidth, style=_funcLineStyle(input_4_LineStyle))
plot(ma5,  title='MA 5',  color=color.new(input_5_color, input_5_LineTransparency), linewidth=input_5_LineWidth, style=_funcLineStyle(input_5_LineStyle))
plot(ma6,  title='MA 6',  color=color.new(input_6_color, input_6_LineTransparency), linewidth=input_6_LineWidth, style=_funcLineStyle(input_6_LineStyle))
plot(ma7,  title='MA 7',  color=color.new(input_7_color, input_7_LineTransparency), linewidth=input_7_LineWidth, style=_funcLineStyle(input_7_LineStyle))
plot(ma8,  title='MA 8',  color=color.new(input_8_color, input_8_LineTransparency), linewidth=input_8_LineWidth, style=_funcLineStyle(input_8_LineStyle))
plot(ma9,  title='MA 9',  color=color.new(input_9_color, input_9_LineTransparency), linewidth=input_9_LineWidth, style=_funcLineStyle(input_9_LineStyle))
plot(ma10, title='MA 10', color=color.new(input_10_color, input_10_LineTransparency), linewidth=input_10_LineWidth, style=_funcLineStyle(input_10_LineStyle))

var label lbl1 = na
if input_1_showLabel and not na(ma1)
    label.delete(lbl1)
    lbl1 := label.new(bar_index, ma1, valFinal_1_MaType + " " + str.tostring(int(valFinal_1_MaLength)) + " " + f_timeframeToLabel(input_1_Timeframe), xloc.bar_index, yloc.price, color.new(input_1_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl2 = na
if input_2_showLabel and not na(ma2)
    label.delete(lbl2)
    lbl2 := label.new(bar_index, ma2, valFinal_2_MaType + " " + str.tostring(int(valFinal_2_MaLength)) + " " + f_timeframeToLabel(input_2_Timeframe), xloc.bar_index, yloc.price, color.new(input_2_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl3 = na
if input_3_showLabel and not na(ma3)
    label.delete(lbl3)
    lbl3 := label.new(bar_index, ma3, valFinal_3_MaType + " " + str.tostring(int(valFinal_3_MaLength)) + " " + f_timeframeToLabel(input_3_Timeframe), xloc.bar_index, yloc.price, color.new(input_3_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl4 = na
if input_4_showLabel and not na(ma4)
    label.delete(lbl4)
    lbl4 := label.new(bar_index, ma4, valFinal_4_MaType + " " + str.tostring(int(valFinal_4_MaLength)) + " " + f_timeframeToLabel(input_4_Timeframe), xloc.bar_index, yloc.price, color.new(input_4_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl5 = na
if input_5_showLabel and not na(ma5)
    label.delete(lbl5)
    lbl5 := label.new(bar_index, ma5, valFinal_5_MaType + " " + str.tostring(int(valFinal_5_MaLength)) + " " + f_timeframeToLabel(input_5_Timeframe), xloc.bar_index, yloc.price, color.new(input_5_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl6 = na
if input_6_showLabel and not na(ma6)
    label.delete(lbl6)
    lbl6 := label.new(bar_index, ma6, valFinal_6_MaType + " " + str.tostring(int(valFinal_6_MaLength)) + " " + f_timeframeToLabel(input_6_Timeframe), xloc.bar_index, yloc.price, color.new(input_6_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl7 = na
if input_7_showLabel and not na(ma7)
    label.delete(lbl7)
    lbl7 := label.new(bar_index, ma7, valFinal_7_MaType + " " + str.tostring(int(valFinal_7_MaLength)) + " " + f_timeframeToLabel(input_7_Timeframe), xloc.bar_index, yloc.price, color.new(input_7_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl8 = na
if input_8_showLabel and not na(ma8)
    label.delete(lbl8)
    lbl8 := label.new(bar_index, ma8, valFinal_8_MaType + " " + str.tostring(int(valFinal_8_MaLength)) + " " + f_timeframeToLabel(input_8_Timeframe), xloc.bar_index, yloc.price, color.new(input_8_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl9 = na
if input_9_showLabel and not na(ma9)
    label.delete(lbl9)
    lbl9 := label.new(bar_index, ma9, valFinal_9_MaType + " " + str.tostring(int(valFinal_9_MaLength)) + " " + f_timeframeToLabel(input_9_Timeframe), xloc.bar_index, yloc.price, color.new(input_9_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

var label lbl10 = na
if input_10_showLabel and not na(ma10)
    label.delete(lbl10)
    lbl10 := label.new(bar_index, ma10, valFinal_10_MaType + " " + str.tostring(int(valFinal_10_MaLength)) + " " + f_timeframeToLabel(input_10_Timeframe), xloc.bar_index, yloc.price, color.new(input_10_color, input_curve_label_transp), label.style_label_left, input_curve_label_color, input_curve_label_size)

//---------------------------------------------------------------------------------------------------------------------}
//ALERTS
//---------------------------------------------------------------------------------------------------------------------{
alertcondition(currentAlerts.internalBullishBOS,        'Internal Bullish BOS',         'Internal Bullish BOS formed')
alertcondition(currentAlerts.internalBullishCHoCH,      'Internal Bullish CHoCH',       'Internal Bullish CHoCH formed')
alertcondition(currentAlerts.internalBearishBOS,        'Internal Bearish BOS',         'Internal Bearish BOS formed')
alertcondition(currentAlerts.internalBearishCHoCH,      'Internal Bearish CHoCH',       'Internal Bearish CHoCH formed')

alertcondition(currentAlerts.swingBullishBOS,           'Bullish BOS',                  'Internal Bullish BOS formed')
alertcondition(currentAlerts.swingBullishCHoCH,         'Bullish CHoCH',                'Internal Bullish CHoCH formed')
alertcondition(currentAlerts.swingBearishBOS,           'Bearish BOS',                  'Bearish BOS formed')
alertcondition(currentAlerts.swingBearishCHoCH,         'Bearish CHoCH',                'Bearish CHoCH formed')

alertcondition(currentAlerts.internalBullishOrderBlock, 'Bullish Internal OB Breakout', 'Price broke bullish internal OB')
alertcondition(currentAlerts.internalBearishOrderBlock, 'Bearish Internal OB Breakout', 'Price broke bearish internal OB')
alertcondition(currentAlerts.swingBullishOrderBlock,    'Bullish Swing OB Breakout',    'Price broke bullish swing OB')
alertcondition(currentAlerts.swingBearishOrderBlock,    'Bearish Swing OB Breakout',    'Price broke bearish swing OB')

alertcondition(currentAlerts.equalHighs,                'Equal Highs',                  'Equal highs detected')
alertcondition(currentAlerts.equalLows,                 'Equal Lows',                   'Equal lows detected')

alertcondition(currentAlerts.bullishFairValueGap,       'Bullish FVG',                  'Bullish FVG formed')
alertcondition(currentAlerts.bearishFairValueGap,       'Bearish FVG',                  'Bearish FVG formed')

//---------------------------------------------------------------------------------------------------------------------}
