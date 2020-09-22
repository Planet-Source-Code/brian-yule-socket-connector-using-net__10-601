<div align="center">

## Socket Connector using \.NET


</div>

### Description

It shows how to connect a socket using .NET assemblies for socket transmissions.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Brian Yule](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/brian-yule.md)
**Level**          |Advanced
**User Rating**    |4.6 (23 globes from 5 users)
**Compatibility**  |VB\.NET
**Category**       |[System Services/ Functions](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/system-services-functions__10-23.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/brian-yule-socket-connector-using-net__10-601/archive/master.zip)





### Source Code

```
Private Class ConnectorDotNet
    Implements System.IDisposable
    Private Class AsyncClientSock
      Private s As System.Net.Sockets.Socket
      Dim inBuffer() As Byte
      Dim State As Object = Nothing
      Public BufferSize As Integer = 1024 ' This is default for server and client.
      Public Event Received(ByVal strRec As String, ByVal bytesReceived As Integer)
      Public Event Connected()
      Public Event SendReady(ByVal BytesSent As Integer)
      Public Event Accepted(ByVal acceptSocket As System.Net.Sockets.Socket)
      Public Event Closed()
      Public Sub New()
        s = New System.Net.Sockets.Socket(System.Net.Sockets.AddressFamily.InterNetwork, System.Net.Sockets.SocketType.Stream, System.Net.Sockets.ProtocolType.Tcp)
      End Sub
      Public ReadOnly Property IsConnected() As Boolean
        Get
          Return s.Connected
        End Get
      End Property
      Public ReadOnly Property RemoteEndPoint() As System.Net.EndPoint
        Get
          Return s.RemoteEndPoint
        End Get
      End Property
      Public Sub Connect(ByVal Address As String, ByVal Port As Integer)
        Dim ipEndPt As System.Net.IPEndPoint
        ipEndPt = New System.Net.IPEndPoint(System.Net.Dns.Resolve(Address).AddressList(0), Port)
        s.BeginConnect(ipEndPt, AddressOf asyncConnected, State)
      End Sub
      Public Sub Send(ByVal Msg As String)
        Dim outBuffer() As Byte
        ReDim outBuffer(System.Text.ASCIIEncoding.ASCII.GetBytes(Msg).Length)
        outBuffer = System.Text.ASCIIEncoding.ASCII.GetBytes(Msg)
        s.BeginSend(outBuffer, 0, outBuffer.Length, System.Net.Sockets.SocketFlags.None, AddressOf asyncSent, State)
      End Sub
      Public Sub Send(ByVal Msg() As Byte)
        s.BeginSend(Msg, 0, Msg.Length, System.Net.Sockets.SocketFlags.None, AddressOf asyncSent, State)
      End Sub
      Public Sub Accept()
        s.BeginAccept(AddressOf asyncAccepted, State)
      End Sub
      Public Sub Close()
        s.Shutdown(System.Net.Sockets.SocketShutdown.Both)
        s.Close()
      End Sub
      Private Sub asyncConnected(ByVal ar As IAsyncResult)
        s.EndConnect(ar)
        ReDim inBuffer(BufferSize)
        s.BeginReceive(inBuffer, 0, inBuffer.Length, System.Net.Sockets.SocketFlags.None, AddressOf asyncRecieved, State)
        RaiseEvent Connected()
      End Sub
      Private Sub asyncSent(ByVal ar As IAsyncResult)
        RaiseEvent SendReady(s.EndSend(ar))
      End Sub
      Private Sub asyncRecieved(ByVal ar As IAsyncResult)
        Dim bytesReceived As Integer
        bytesReceived = s.EndReceive(ar)
        If bytesReceived > 0 Then
          RaiseEvent Received(System.Text.ASCIIEncoding.ASCII.GetString(inBuffer), bytesReceived)
          ReDim inBuffer(BufferSize)
          s.BeginReceive(inBuffer, 0, inBuffer.Length, System.Net.Sockets.SocketFlags.None, AddressOf asyncRecieved, State)
        ElseIf bytesReceived = 0 Then
          Call s.Shutdown(System.Net.Sockets.SocketShutdown.Both)
          s.Close()
          RaiseEvent Closed()
        End If
      End Sub
      Private Sub asyncAccepted(ByVal ar As IAsyncResult)
        RaiseEvent Accepted(s.EndAccept(ar))
      End Sub
      Protected Overrides Sub Finalize()
        MyBase.Finalize()
      End Sub
    End Class
    Public Event Recieve(ByRef Message() As String)
    Public Event Connected()
    Public Event Error_Renamed()
    Public Event Closed()
    Public Debugging As Boolean
    Private WithEvents wscControl As AsyncClientSock
    Public Overloads Sub Dispose() Implements System.IDisposable.Dispose
      wscControl = Nothing
    End Sub
    Public Sub New()
      MyBase.New()
      wscControl = New AsyncClientSock()
    End Sub
    Protected Overrides Sub Finalize()
      MyBase.Finalize()
    End Sub
    Private Sub LogMessage(ByRef Message() As String)
      Dim FreeFileNo, i As Short
      Dim strMessage As String
      FreeFileNo = FreeFile()
      FileOpen(FreeFileNo, "IO.log", OpenMode.Append)
      For i = 0 To UBound(Message)
        strMessage = strMessage & Message(i) & ";"
      Next i
      Console.WriteLine(Now & vbTab & strMessage)
      PrintLine(FreeFileNo, Now, strMessage)
      FileClose(FreeFileNo)
    End Sub
    Public Sub Send(ByRef Message() As String)
      On Error Resume Next
      Dim i As Int16
      Dim strSend As String
      For i = LBound(Message) To UBound(Message)
        If Message(i) = "" Then
          strSend = strSend & "-1;"
        Else
          strSend = strSend & Message(i) & ";"
        End If
      Next i
      Call wscControl.Send(strSend)
    End Sub
    Public Sub Connect(ByVal Address As String, ByVal Port As Integer)
      On Error Resume Next
      Call wscControl.Connect(Address, Port)
    End Sub
    Private Sub GetMessages(ByVal strMessage As String, ByRef Messages() As String, ByVal Delimiter As String)
      On Error Goto ErrorHandler
      Dim StartPos, MessageCounter As Short
      StartPos = 1
      Do While InStr(StartPos, strMessage, Delimiter)
        ReDim Preserve Messages(MessageCounter)
        Messages(MessageCounter) = Trim(Mid(strMessage, StartPos, InStr(StartPos, strMessage, Delimiter) - StartPos))
        StartPos = InStr(StartPos, strMessage, Delimiter) + 1
        MessageCounter = MessageCounter + 1
      Loop
      If Right(strMessage, Len(strMessage) - StartPos + 1) <> "" Then
        ReDim Preserve Messages(MessageCounter)
        Messages(MessageCounter) = Trim(Right(strMessage, Len(strMessage) - StartPos + 1))
      End If
      Exit Sub
ErrorHandler:
      Console.WriteLine("Connector.GetMessages")
    End Sub
    Private Sub wscControl_DataArrival(ByVal strRec As String, ByVal bytesReceived As Integer) Handles wscControl.Received
      On Error Resume Next
      Dim strBuffer As String
      Dim i As Short
      strBuffer = strRec
      Dim Messages() As String
      Call GetMessages(strBuffer, Messages, Chr(10))
      If Debugging Then Call LogMessage(Messages)
      Dim strArgArray() As String
      For i = 0 To UBound(Messages)
        If Messages(i) <> "" And Messages(i) <> ControlChars.CrLf Then
          Call GetMessages(Messages(i), strArgArray, ";")
          RaiseEvent Recieve(strArgArray)
        End If
      Next i
    End Sub
    Private Sub wscControl_Connected() Handles wscControl.Connected
      On Error Resume Next
      RaiseEvent Connected()
    End Sub
    Private Sub wscControl_Closed() Handles wscControl.Closed
      On Error Resume Next
      RaiseEvent Closed()
    End Sub
  End Class
```

