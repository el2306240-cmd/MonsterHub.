function love.load()
    love.window.setTitle("Monster Hub")
    love.window.setMode(800, 600)

    fonte = love.graphics.newFont(32)
    fontePequena = love.graphics.newFont(18)
    fonteMini = love.graphics.newFont(14)

    botaoAberto = true
    botaoLargura = 120
    botaoAltura = 50
    botaoX = 800 - botaoLargura - 20
    botaoY = 20

    quadLargura = 250
    quadAltura = 400
    quadRaioBorda = 25
    quadX = (800 - quadLargura) / 2
    quadY = (600 - quadAltura) / 2

    inicioLargura = 120
    inicioAltura = 30
    inicioX = quadX + 10
    inicioY = quadY + 10
    opcoesVisiveis = false

    xrayAtivado = false
    circRaio = 50
    circX = 100
    circY = 100
    arrastando = false

    dragaoImg = love.graphics.newImage("dragon.png")

    inimigos = {
        {x = quadX + 50, y = quadY + 50, raio = 10},
        {x = quadX + 150, y = quadY + 80, raio = 10},
        {x = quadX + 120, y = quadY + 150, raio = 10}
    }

    paredes = {
        {x = quadX + 30, y = quadY + 40, w = 50, h = 100},
        {x = quadX + 100, y = quadY + 20, w = 60, h = 50},
        {x = quadX + 140, y = quadY + 100, w = 40, h = 80}
    }

    -- Barra de velocidade
    sliderX, sliderY = inicioX, inicioY + 130
    sliderLargura, sliderAltura = 150, 8
    sliderValor = 0
    sliderMax = 500
    arrastandoSlider = false
    velocidadeAtivada = false

    -- Campo e botão de pulo
    inputPulo = ""
    ativoInput = false
    puloAltura = 0
end

function love.draw()
    love.graphics.setFont(fonte)
    love.graphics.setColor(1, 1, 1)
    love.graphics.print("Monster Hub", 20, 20)

    if botaoAberto then
        love.graphics.setColor(0, 1, 0)
    else
        love.graphics.setColor(1, 0, 0)
    end
    love.graphics.rectangle("fill", botaoX, botaoY, botaoLargura, botaoAltura, 10, 10)
    love.graphics.setColor(1, 1, 1)
    love.graphics.print(botaoAberto and "Aberto" or "Fechado", botaoX + 10, botaoY + 10)

    if botaoAberto then
        -- Quadrado com sombra
        love.graphics.setColor(0, 0, 0, 0.4)
        love.graphics.rectangle("fill", quadX + 8, quadY + 8, quadLargura, quadAltura, quadRaioBorda, quadRaioBorda)
        love.graphics.setColor(1, 0, 0)
        love.graphics.rectangle("fill", quadX, quadY, quadLargura, quadAltura, quadRaioBorda, quadRaioBorda)

        -- Botão "Início"
        love.graphics.setFont(fontePequena)
        love.graphics.setColor(0.2, 0.6, 1)
        love.graphics.rectangle("fill", inicioX, inicioY, inicioLargura, inicioAltura, 5, 5)
        love.graphics.setColor(1, 1, 1)
        love.graphics.print("Início", inicioX + 10, inicioY + 5)

        if opcoesVisiveis then
            -- Opção Xray
            local opcY = inicioY + 40
            love.graphics.setColor(0.1, 0.1, 0.1)
            love.graphics.rectangle("fill", inicioX, opcY, inicioLargura, 30, 5, 5)
            love.graphics.setColor(1, 1, 1)
            love.graphics.print("Xray", inicioX + 10, opcY + 5)
            love.graphics.setFont(fonteMini)
            love.graphics.setColor(1, 1, 1, 0.4)
            love.graphics.print("Vê oponentes através da parede", inicioX, opcY + 25)

            -- Slider de velocidade
            local sliderYLocal = opcY + 70
            love.graphics.setFont(fontePequena)
            love.graphics.setColor(1, 1, 1)
            love.graphics.print("Velocidade: " .. math.floor(sliderValor), sliderX, sliderYLocal - 25)
            love.graphics.setColor(0.3, 0.3, 0.3)
            love.graphics.rectangle("fill", sliderX, sliderYLocal, sliderLargura, sliderAltura, 4, 4)
            love.graphics.setColor(0.2, 0.8, 1)
            love.graphics.circle("fill", sliderX + (sliderValor / sliderMax) * sliderLargura, sliderYLocal + sliderAltura / 2, 8)

            -- Botão ativar velocidade
            local botaoVelocidadeY = sliderYLocal + 30
            love.graphics.setColor(0.2, 0.8, 0.2)
            love.graphics.rectangle("fill", sliderX, botaoVelocidadeY, 150, 30, 5, 5)
            love.graphics.setColor(1, 1, 1)
            love.graphics.print("Ativar Velocidade", sliderX + 10, botaoVelocidadeY + 5)
            love.graphics.setFont(fonteMini)
            love.graphics.setColor(1, 1, 1, 0.4)
            love.graphics.print("Ativa a velocidade de acordo com o número", sliderX, botaoVelocidadeY + 35)

            -- Campo de texto do pulo
            local puloY = botaoVelocidadeY + 70
            love.graphics.setColor(0.9, 0.9, 0.9)
            love.graphics.rectangle("fill", sliderX, puloY, 150, 30, 5, 5)
            love.graphics.setColor(0, 0, 0)
            love.graphics.print(inputPulo, sliderX + 10, puloY + 5)

            -- Botão "Ativar Pulo"
            local botaoPuloY = puloY + 40
            love.graphics.setColor(0.2, 0.8, 0.4)
            love.graphics.rectangle("fill", sliderX, botaoPuloY, 150, 30, 5, 5)
            love.graphics.setColor(1, 1, 1)
            love.graphics.print("Ativar Pulo", sliderX + 20, botaoPuloY + 5)
            love.graphics.setFont(fonteMini)
            love.graphics.setColor(1, 1, 1, 0.4)
            love.graphics.print("Define a altura do pulo conforme o número digitado", sliderX, botaoPuloY + 35)
        end

        -- Paredes e inimigos
        for _, parede in ipairs(paredes) do
            love.graphics.setColor(0.5, 0.5, 0.5)
            love.graphics.rectangle("fill", parede.x, parede.y, parede.w, parede.h)
        end
        for _, inimigo in ipairs(inimigos) do
            if xrayAtivado then
                love.graphics.setColor(0, 1, 1, 0.7)
                love.graphics.circle("fill", inimigo.x, inimigo.y, inimigo.raio + 2)
            else
                local bloqueado = false
                for _, parede in ipairs(paredes) do
                    if inimigo.x > parede.x and inimigo.x < parede.x + parede.w and
                       inimigo.y > parede.y and inimigo.y < parede.y + parede.h then
                        bloqueado = true
                    end
                end
                if not bloqueado then
                    love.graphics.setColor(0, 1, 0)
                    love.graphics.circle("fill", inimigo.x, inimigo.y, inimigo.raio)
                end
            end
        end
    else
        -- Círculo minimizado com dragão
        love.graphics.setColor(0, 0, 0, 0.4)
        love.graphics.circle("fill", circX + 4, circY + 4, circRaio)
        love.graphics.setColor(1, 1, 1)
        love.graphics.draw(dragaoImg, circX - dragaoImg:getWidth()/2, circY - dragaoImg:getHeight()/2)
    end
end

function love.mousepressed(mx, my, button)
    if button == 1 then
        -- abrir/fechar
        if mx >= botaoX and mx <= botaoX + botaoLargura and my >= botaoY and my <= botaoY + botaoAltura then
            botaoAberto = not botaoAberto
        end

        if botaoAberto then
            if mx >= inicioX and mx <= inicioX + inicioLargura and my >= inicioY and my <= inicioY + inicioAltura then
                opcoesVisiveis = not opcoesVisiveis
            end

            -- Clicar no Xray
            local opcY = inicioY + 40
            if opcoesVisiveis and mx >= inicioX and mx <= inicioX + inicioLargura and my >= opcY and my <= opcY + 30 then
                xrayAtivado = not xrayAtivado
            end

            -- Slider
            local sliderYLocal = opcY + 70
            if mx >= sliderX and mx <= sliderX + sliderLargura and my >= sliderYLocal - 5 and my <= sliderYLocal + 15 then
                arrastandoSlider = true
            end

            -- Botão ativar velocidade
            local botaoVelocidadeY = sliderYLocal + 30
            if mx >= sliderX and mx <= sliderX + 150 and my >= botaoVelocidadeY and my <= botaoVelocidadeY + 30 then
                velocidadeAtivada = true
            end

            -- Campo de texto do pulo
            local puloY = botaoVelocidadeY + 70
            ativoInput = mx >= sliderX and mx <= sliderX + 150 and my >= puloY and my <= puloY + 30

            -- Botão ativar pulo
            local botaoPuloY = puloY + 40
            if mx >= sliderX and mx <= sliderX + 150 and my >= botaoPuloY and my <= botaoPuloY + 30 then
                local num = tonumber(inputPulo)
                if num then
                    if num < 0 then num = 0 elseif num > 100 then num = 100 end
                    puloAltura = num
                    print("Pulo ativado com altura:", puloAltura)
                end
            end
        else
            -- arrastar dragão
            local dx, dy = mx - circX, my - circY
            if dx*dx + dy*dy <= circRaio*circRaio then
                arrastando = true
                offsetX, offsetY = dx, dy
            end
        end
    end
end

function love.mousereleased(_, _, button)
    if button == 1 then
        arrastando = false
        arrastandoSlider = false
    end
end

function love.mousemoved(mx, _, dx, _)
    if arrastando then
        circX = mx - offsetX
        circY = circY
    end
    if arrastandoSlider then
        local rel = mx - sliderX
        sliderValor = math.max(0, math.min(sliderMax, (rel / sliderLargura) * sliderMax))
    end
end

function love.textinput(t)
    if ativoInput then
        if #inputPulo < 3 and tonumber(t) ~= nil then
            inputPulo = inputPulo .. t
        end
    end
end

function love.keypressed(key)
    if ativoInput then
        if key == "backspace" then
            inputPulo = inputPulo:sub(1, -2)
        elseif key == "return" then
            ativoInput = false
        end
    end
end
