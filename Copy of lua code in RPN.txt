screen = platform.window
entry = ""
local stack = {}
local stackSize = 200
local isOnEntryLine = false
local debug = false
local debugChar = ""
local originalSettings = math.getEvalSettings()

--Init
function on.activate()
    for i=1,stackSize do
        stack[i]="0"
    end
end

--Draw
function on.paint(gc) 
    local sh = gc:getStringHeight(entry)
    local sw = gc:getStringWidth("> "..entry)
    local entryLineOffset = 0;
    local alignmentOffset = screen:height()/sh-1 >= 10 and gc:getStringWidth("0") or 0
    
    if isOnEntryLine then 
        gc:drawString("> "..entry, alignmentOffset, screen:height()-sh)
        entryLineOffset = 1;
    end
    
    math.setEvalSettings({ {'Calculation Mode', 'Approximate'} })
    gc:drawString("X:"..stack[1], alignmentOffset, screen:height()-(1+entryLineOffset)*sh)
    gc:drawString("Y:"..stack[2], alignmentOffset, screen:height()-(2+entryLineOffset)*sh)
    gc:drawString("Z:"..stack[3], alignmentOffset, screen:height()-(3+entryLineOffset)*sh)
    for k=4,screen:height()/sh-1 do
        gc:drawString(k..":"..stack[k], k<10 and alignmentOffset or 0, screen:height()-(k+entryLineOffset)*sh)
    end
    if debug then gc:drawString(debugChar, screen:width() - gc:getStringWidth(debugChar), screen:height()-sh) end
    
end 

--Read Chars
function on.charIn(char)
    local variables = {}
    local singleLine = false
    local validOperation = false
    
    --make entry line independent copy of stack
    if isOnEntryLine then
        variables[1]=entry
        variables[2]=stack[1]
    else
        variables[1]=stack[1]
        variables[2]=stack[2]
    end
    if char == "°" then variables[1] = math.rad(variables[1]) end
    
    debugChar = char..type(char)
    print(char)
    
    --Is it a handelled operation
    if char == "−" then variables[1] = math.evalStr("-"..variables[1]);   singleLine = true;     validOperation = true
    elseif char == "+" then variables[2] = math.evalStr(variables[2].."+"..variables[1]);  validOperation = true
    elseif char == "-" then variables[2] = math.evalStr(variables[2].."-"..variables[1]);  validOperation = true
    elseif char == "*" then variables[2] = math.evalStr(variables[2].."*"..variables[1]);  validOperation = true
    elseif char == "/" then variables[2] = math.evalStr(variables[2].."/"..variables[1]);  validOperation = true
    elseif char == "^" then variables[2] = math.evalStr(variables[2].."^"..variables[1]);  validOperation = true
    elseif char == "root(" then variables[2] = math.evalStr("root("..variables[1]..", "..variables[2]..")");  validOperation = true
    elseif char == "√(" then variables[1] = math.evalStr("√("..variables[1]..")");    singleLine = true;  validOperation = true
    elseif char == "ln(" then variables[1] = math.evalStr("ln("..variables[1]..")");    singleLine = true;  validOperation = true
    elseif char == "log(" then variables[2] = math.evalStr("log("..variables[1]..", "..variables[2]..")");  validOperation = true
    elseif char == "^2" then variables[1] = math.evalStr(variables[1].."^2");    singleLine = true;   validOperation = true
    elseif char == "exp(" then variables[1] = math.evalStr("exp("..variables[1]..")");    singleLine = true;   validOperation = true
    elseif char == "sin(" then variables[1] = math.evalStr("sin("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "cos(" then variables[1] = math.evalStr("cos("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "tan(" then variables[1] = math.evalStr("tan("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "csc(" then variables[1] = math.evalStr("csc("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "sec(" then variables[1] = math.evalStr("sec("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "cot(" then variables[1] = math.evalStr("cot("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "sin(" then variables[1] = math.evalStr("sin("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "cos(" then variables[1] = math.evalStr("cos("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "tan(" then variables[1] = math.evalStr("tan("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "csc(" then variables[1] = math.evalStr("csc("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "sec(" then variables[1] = math.evalStr("sec("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "cot(" then variables[1] = math.evalStr("cot("..variables[1]..")");   singleLine = true;   validOperation = true
    elseif char == "10^(" then variables[2] = math.evalStr("10^"..variables[1]);   singleLine = true;   validOperation = true
    end
    
    --push updates back to stack
    if isOnEntryLine then
        entry=variables[1]
        stack[1]=variables[2]
    else
        stack[1]=variables[1]
        stack[2]=variables[2]
    end
    
    --pop stack if operation consumed a line
    if validOperation and isOnEntryLine == false and singleLine == false then popStack() end
    if validOperation and singleLine == false  then isOnEntryLine = false; entry = "" end
    
    --Is it a number and not an operation (some operations appear as numbers)
    if validOperation == false and (type(tonumber(char)) == "number" or char == "." or char == "" or char == "" or char == "°") then --blank is "EE", "" is i (complex variable)
        entry = entry..char
        isOnEntryLine = true
    --handle constants (not an operation or a number
    elseif char == "π" then entry = entry..math.pi; isOnEntryLine = true --Pi
    elseif char == "" then entry = entry..math.exp(1); isOnEntryLine = true end --e
    
    -- Refresh the screen after each key is pressed.
    screen:invalidate()
end

function popStack()
    local poppedValue = stack[1]
    for i=1,stackSize do
        stack[i]=stack[i+1]
    end
    stack[stackSize] = ""
    return poppedValue
end

--Push
function on.enterKey()
    for i=0,stackSize-1 do
        stack[stackSize-i]=stack[stackSize-i-1]
    end
    if isOnEntryLine then
        stack[1] = entry
        entry = ""
    else
        stack[1] = stack[2]
    end
    isOnEntryLine = false
    screen:invalidate()
end

--Pop
function on.returnKey()
    popStack()
    screen:invalidate()
end

--Delete characters from entry
function on.backspaceKey()
    entry = entry:usub(0,-2)
    screen:invalidate()
end

--Swap
function on.tabKey()
    local temp = stack[1]
    stack[1] = stack[2]
    stack[2]=temp
    screen:invalidate()
end