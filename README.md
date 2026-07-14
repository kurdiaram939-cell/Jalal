local isSearchedCards = false
local cardsResults = {}
local configCards = {
    [1] = {name="کارتی برۆنز", hex={1918976790, 1348420452, 829121377, 0, 0, 0}},
    [2] = {name="کارتی سەوز", hex={1918976790, 1348420452, 845898593, 0, 0, 0}},
    [3] = {name="کارتی شین", hex={1918976790, 1348420452, 862675809, 0, 0, 0}},
    [4] = {name="کارتی مۆر", hex={1918976790, 1348420452, 879453025, 0, 0, 0}},
    [5] = {name="کارتی گۆڵد", hex={1918976790, 1348420452, 896230241, 0, 0, 0}}
}

function CardsSystemAram()
    gg.setVisible(false)
    local menu = gg.multiChoice({
       "╔═══════ 🦋════════╗\nꕤ  🟤       کیسە کارتی بڕۆنز         ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🟢      کیسە کارتی سەووز       ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🔵        کیسەکارتی شین          ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🟣        کیسە کارتی مۆر          ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🟡       کیسە کارتی گۆڵد          ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🔄          گــــەڕانەوە                ꕤ\n╚═════════════════╝",
       "╔═══════ 🦋════════╗\nꕤ  🚪          دەرچـــــوون              ꕤ\n╚═════════════════╝",
    }, nil, "╔══════════════════════╗\n    🦋 🄰🅁🄰🄼 🄺🅄🅁🄳 🅃🄾🅆🄽 🦋\n╚══════════════════════╝")
    
    if menu == nil then return CardsSystemAram() end

    if menu[6] then 
        isSearchedCards = false
        cardsResults = {}
        gg.clearResults()
        gg.clearList()
        gg.toast("🦋 🅰︎🆁︎🅰︎🅼︎🅺︎🆄︎🆁︎🅳︎🆃︎🅾︎🆆︎🅽︎🦋")
        if MainMenu then return MainMenu() end 
        return 
    end

    if menu[7] then gg.clearList(); gg.clearResults(); os.exit() end
    
    local anySelected = false
    for i=1, 5 do if menu[i] then anySelected = true break end end
    if not anySelected then return CardsSystemAram() end

    if not isSearchedCards then
        gg.clearResults()
        -- ڕاستکردنەوەی ڕەینجەکان بۆ ئەندرۆید ١٦
        gg.setRanges(gg.REGION_ANONYMOUS | gg.REGION_C_ALLOC | gg.REGION_CODE_APP)
        gg.toast("🔍 خەریکی گەڕانە بۆ کارتەکان...")
        
        -- ڕاستکردنەوەی flags بۆ TYPE_DWORD
        gg.searchNumber("65537~65542;1970225964;29::457", gg.TYPE_DWORD, false, gg.SIGN_EQUAL, 0, -1)
        gg.refineNumber("29", gg.TYPE_DWORD, false, gg.SIGN_EQUAL, 0, -1)
        
        local count = gg.getResultCount()
        if count == 0 then 
            isSearchedCards = false 
            gg.alert("❌ دڵنیابەوە لە یاری بە سراوەتەوە بە جێم ❌") 
            return CardsSystemAram()
        end
        if count > 500 then count = 500 end -- ڕێگری لە کراش
        cardsResults = gg.getResults(count)
        isSearchedCards = true
    end

    local input = gg.prompt({'بڕی پێویست بنوسە:'}, {'0'}, {'number'})
    if input == nil then return CardsSystemAram() end

    local edit = {}
    local freeze = {}

    local slotIdx = 1
    for i = 1, 5 do
        if menu[i] and cardsResults[slotIdx] then
            local v = configCards[i]
            local r = cardsResults[slotIdx]
            
            table.insert(freeze, {address = r.address + 12, value = 2, flags = gg.TYPE_DWORD, freeze = true})
            for j, h in ipairs(v.hex) do 
                table.insert(edit, {address = r.address + 12 + (j * 4), value = h, flags = gg.TYPE_DWORD}) 
            end
            table.insert(edit, {address = r.address + 40, value = 0, flags = gg.TYPE_DWORD})
            table.insert(edit, {address = r.address + 44, value = tonumber(input[1]), flags = gg.TYPE_DWORD})
            
            slotIdx = slotIdx + 1
        end
    end
    
    if #edit > 0 then
        gg.setValues(edit) 
        gg.addListItems(freeze)
        gg.toast("✅ کارتەکان جێگیر کران")
        gg.setVisible(false)
        while not gg.isVisible() do gg.sleep(200) end
        return CardsSystemAram() 
    else
        gg.alert("❌ هیچ گۆڕانکارییەک نەکرا!")
        return CardsSystemAram() 
    end
end
