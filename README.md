# net-transfer
Object for conveniently sending and receiving large data/files via `net`.

## Code
Written in [Yuescript](https://github.com/pigpigyyy/Yuescript), compiled in Lua code can be found in [releases](https://github.com/PrikolMen/net-transfer/releases).

## Global Functions
- `NetTransferObject` NetTransfer( `string` transferName, `boolean` verifyChecksums, `boolean` unreliable ) - Creates a data transfer object with the specified parameters.

## Meta Functions
- `number` NetTransfer:GetTransmissionSpeed() - Returns network messages per second.
- `nil` NetTransfer:SetTransmissionSpeed( `number` packet per second, `boolean` noCeil ) - Sets network messages per second.
- `boolean` NetTransfer:IsUnreliable() - https://wiki.facepunch.com/gmod/net.Start
- `nil` NetTransfer:SetUnreliable( `boolean` value ) - https://wiki.facepunch.com/gmod/net.Start
- `boolean` NetTransfer:IsVerifyChecksums() - Returns `true` if network message contains and verifies the checksum
- `nil` NetTransfer:SetVerifyChecksums( `boolean` value ) - Whether the network message must contain and verify the checksum.
- `any` NetTransfer:GetFilter() - Returns the filter of allowed players to send and receive.
- `nil` NetTransfer:SetFilter( `boolean/number/string/table/function/entity/vector/RecipientFilter` filter ) - Sets the filter of allowed players to send and receive
- `boolean` NetTransfer:IsAllowedPlayer( `Player` ply ) - Checks if the player is allowed by the filter.
- `boolean` NetTransfer:IsCompressedData() - returns `true` if the data is compressed.
- `string` NetTransfer:CompressData() - Compresses current data using LZMA.
- `string` NetTransfer:DecompressData() - Decompresses current data from LZMA.
- `string` NetTransfer:GetTransmittedData() - Returns the current data, if compressed returns the compressed data.
- `string` NetTransfer:SetTransmittedData( `string` transmittedData, `boolean` compress, `boolean` decompress ) - Sets data and compresses or decompresses it as needed.
- `boolean` NetTransfer:IsSending() - Returns `true` if data is being sent at the moment.
- `nil` NetTransfer:Send( `Player/RecipientFilter/table` target ) - Sends the current data to the selected target.
- `boolean` NetTransfer:IsReceiving() - Returns `true` if data receiving is in progress.
- `nil` NetTransfer:Receive( `function` func, `boolean` permanent ) - As soon as the data is completely retrieved it will execute the specified function and return the data in it (if the data was compressed it will decompress it), by default the function is executed once after it is deleted unless permanent is set to `true`.
- `nil` NetTransfer:OnProgress( `function` func ) - Sets the function that will receive the progress of data retrieval.
- `nil` NetTransfer:Finish() - Forcibly terminates the data transfer.
- `nil` NetTransfer:Remove() - Removes an object from the list to receive data.

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

![image](https://github.com/PrikolMen/net-transfer/assets/44779902/4e1cb05a-696a-4ad7-8e8e-ba11f593d45b)
![image](https://github.com/PrikolMen/net-transfer/assets/44779902/37db7643-c3c0-4b1b-9a1d-f80576b9e3ab)
