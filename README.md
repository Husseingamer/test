Imports System.Collections.Generic
Imports System.Collections.ObjectModel
Imports System.Linq
Imports System.Text
Imports System.Threading
Imports System.Threading.Tasks
Imports System.Windows
Imports System.Windows.Controls
Imports System.Windows.Data
Imports System.Windows.Documents
Imports System.Windows.Input
Imports System.Windows.Media
Imports System.Windows.Media.Animation
Imports System.Windows.Media.Imaging
Imports System.Windows.Navigation
Imports System.Windows.Shapes
Imports Zlo.Extras
Imports ZloGUILauncher.Servers

Namespace ZloGUILauncher.Views
	''' <summary>
	''' Interaction logic for BF4ServerListView.xaml
	''' </summary>
	Public Partial Class BF4ServerListView
		Inherits UserControl
		Public ReadOnly Property ViewSource() As CollectionViewSource
			Get
				Return TryCast(TryFindResource("ServersView"), CollectionViewSource)
			End Get
		End Property
		Private m_BF4_Servers As ObservableCollection(Of BF4_GUI_Server)
		Public ReadOnly Property BF4_GUI_Servers() As ObservableCollection(Of BF4_GUI_Server)
			Get
				If m_BF4_Servers Is Nothing Then
					m_BF4_Servers = New ObservableCollection(Of BF4_GUI_Server)()
				End If
				Return m_BF4_Servers
			End Get
		End Property
		Public ReadOnly Property DataServersList() As API_BF4ServersListBase
			Get
				Return App.Client.BF4Servers
			End Get
		End Property

		Public Sub New()
			InitializeComponent()
			AddHandler DataServersList.ServerAdded, AddressOf DataServersList_ServerAdded
			AddHandler DataServersList.ServerUpdated, AddressOf DataServersList_ServerUpdated
			AddHandler DataServersList.ServerRemoved, AddressOf DataServersList_ServerRemoved

			ViewSource.Source = BF4_GUI_Servers
		End Sub
		Private Sub DataServersList_ServerRemoved(id As UInteger, server As API_BF4ServerBase)
			If server IsNot Nothing Then
				Dispatcher.Invoke(Function() 
				'remove from current list
				Dim ser = BF4_GUI_Servers.Find(Function(s) s.ID = id)
				If ser IsNot Nothing Then
					BF4_GUI_Servers.Remove(ser)
				End If

End Function)
			End If
		End Sub
		Private Sub DataServersList_ServerUpdated(id As UInteger, server As API_BF4ServerBase)
			Dispatcher.Invoke(Function() 
			Dim equi = BF4_GUI_Servers.Find(Function(x) x.raw = server)
			If equi IsNot Nothing Then
				equi.UpdateAllProps()
				AnimateRow(equi)
			End If

End Function)
		End Sub
		Private Sub DataServersList_ServerAdded(id As UInteger, server As API_BF4ServerBase)
			Dispatcher.Invoke(Function() 
			Dim newserv = New BF4_GUI_Server(server)
			BF4_GUI_Servers.Add(newserv)
			AnimateRow(newserv)

End Function)
		End Sub

		Public Sub AnimateRow(element As BF4_GUI_Server)
			Dim row = TryCast(ServersDG.ItemContainerGenerator.ContainerFromItem(element), DataGridRow)
			If row Is Nothing Then
				Return
			End If



			Dim switchOnAnimation As New ColorAnimation() With { _
				Key .From = Colors.White, _
				Key .[To] = Colors.Pink, _
				Key .Duration = TimeSpan.FromSeconds(1), _
				Key .AutoReverse = True _
			}
			Dim blinkStoryboard As New Storyboard()


			blinkStoryboard.Children.Add(switchOnAnimation)
			Storyboard.SetTargetProperty(switchOnAnimation, New PropertyPath("Background.Color"))
			'animate changed server
			Storyboard.SetTarget(switchOnAnimation, row)

			row.BeginStoryboard(blinkStoryboard)
		End Sub

		Private Sub JoinButton_Click(sender As Object, e As RoutedEventArgs)
			Dim b = TryCast(sender, Button)
			Dim server = DirectCast(b.DataContext, BF4_GUI_Server)
			If server.IsHasPW Then
				Dim inb As New InputBox(requestmsg)
				Dim ish = inb.ShowDialog()
				If ish.HasValue AndAlso ish.Value Then
					Dim pw = inb.OutPut
					App.Client.JoinOnlineGameWithPassWord(OnlinePlayModes.BF4_Multi_Player, server.ID, pw)
				End If
			Else
				App.Client.JoinOnlineGame(OnlinePlayModes.BF4_Multi_Player, server.ID)
			End If
		End Sub
		Private requestmsg As String = "Please Enter the server password : " & vbLf & "Note : If you are sure the server doesn't have a password, press done and leave the password box empty"
		Private Sub JoinSpectatorButton_Click(sender As Object, e As RoutedEventArgs)
			Dim b = TryCast(sender, Button)
			Dim server = DirectCast(b.DataContext, BF4_GUI_Server)
			If server.IsHasPW Then
				Dim inb As New InputBox(requestmsg)
				Dim ish = inb.ShowDialog()
				If ish.HasValue AndAlso ish.Value Then
					Dim pw = inb.OutPut
					App.Client.JoinOnlineGameWithPassWord(OnlinePlayModes.BF4_Spectator, server.ID, pw)
				End If
			Else
				App.Client.JoinOnlineGame(OnlinePlayModes.BF4_Spectator, server.ID)
			End If
		End Sub
		Private Sub JoinCommanderButton_Click(sender As Object, e As RoutedEventArgs)
			Dim b = TryCast(sender, Button)
			Dim server = DirectCast(b.DataContext, BF4_GUI_Server)
			If server.IsHasPW Then
				Dim inb As New InputBox(requestmsg)
				Dim ish = inb.ShowDialog()
				If ish.HasValue AndAlso ish.Value Then
					Dim pw = inb.OutPut
					App.Client.JoinOnlineGameWithPassWord(OnlinePlayModes.BF4_Commander, server.ID, pw)
				End If
			Else
				App.Client.JoinOnlineGame(OnlinePlayModes.BF4_Commander, server.ID)
			End If
		End Sub




		Private Sub ScrollViewer_PreviewMouseWheel(sender As Object, e As MouseWheelEventArgs)
			If sender.[GetType]() = GetType(ScrollViewer) Then
				Dim scrollviewer As ScrollViewer = TryCast(sender, ScrollViewer)
				If e.Delta > 0 Then
					scrollviewer.LineLeft()
				Else
					scrollviewer.LineRight()
				End If
				e.Handled = True
			Else
				Dim d = TryCast(sender, DependencyObject)
				For i As Integer = 0 To VisualTreeHelper.GetChildrenCount(d) - 1
					If TypeOf VisualTreeHelper.GetChild(d, i) Is ScrollViewer Then
						Dim scroll As ScrollViewer = DirectCast(VisualTreeHelper.GetChild(d, i), ScrollViewer)
						If e.Delta > 0 Then
							scroll.LineLeft()
						Else
							scroll.LineRight()
						End If
						e.Handled = True
					End If
				Next
			End If
		End Sub
	End Class
End Namespace
