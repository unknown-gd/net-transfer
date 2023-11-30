# net-transfer
A class for sending binary data through the net library in a simple and fast ( as fast as possible ) way.

## Example Code ( very lazy example )
- SERVER
```lua
concommand.Add( "fs_test_req", function( ply, _, args )
    path = args[1]
    if #path == 0 then
        return
    end

    local fs = NetTransfer( "example" )
    fs:SetTransmittedData( file.Read( path, "GAME" ), true, false )
    fs:OnProgress( function( _, fraction, bytes )
        print( string.format( "Net transfer: %d%% (%d Bytes)", fraction * 100, bytes ) )
    end )

    fs:Send( ply )
end )
```
- CLIENT
```lua
concommand.Add( "fs_test", function()
    local frame = vgui.Create( "DFrame" )
    frame:SetSize( 256, 128 )
    frame:Center()
    frame:MakePopup()
    frame:SetTitle( "Net transfer example" )

    local pbar = frame:Add( "DProgress" )
    pbar:Dock( BOTTOM )

    local button = frame:Add( "DButton" )
    button:SetText( "Start" )
    button:Dock( BOTTOM )

    local text = frame:Add( "DTextEntry" )
    text:Dock( BOTTOM )

    local pbarText = frame:Add( "DLabel" )
    pbarText:Dock( FILL )

    local nt = NetTransfer( "example" )
    local startTime = 0
    nt:Receive( function( _, data )
        pbarText:SetText( string.format( "Total received: %d KB, took %f seconds.", #data / 1024, SysTime() - startTime ) )
        button:SetText( "Start" )
        button:SetEnabled( true )
    end )

    nt:OnProgress( function( _, fraction, bytes )
        if fraction < 1 then
            button:SetText( "Stop" )
            button:SetEnabled( true )
        end

        print( string.format( "Net transfer: %d%% (%d Bytes)", fraction * 100, bytes ), SysTime() - startTime )
        pbarText:SetText( string.format( "Downloaded: %d%%, took %d seconds.", fraction * 100, SysTime() - startTime ) )
        pbar:SetFraction( fraction )
    end )

    function button:DoClick()
        if nt:IsReceiving() then
            self:SetText( "Start" )
            self:SetEnabled( true )
            nt:Finish()
        else
            RunConsoleCommand( "fs_test_req", text:GetValue() )
            self:SetEnabled( false )
            startTime = SysTime()
        end
    end
end )
```
