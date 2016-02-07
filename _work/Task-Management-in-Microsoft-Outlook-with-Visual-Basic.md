---
layout: article
title: "Task Management in Microsoft Outlook with Visual Basic"
date: 2014-06-05 08:01:00
cover: /work/img/Task-Management-in-Microsoft-Outlook-with-Visual-Basic-Cover.png
collection: work
tags:
  - all
  - code
  - visualbasic
  - outlook
---

Visual Basic for Applications (Macro) code to count tasks in Microsoft Outlook.

<!--more-->

![Task Management in Microsoft Outlook with Visual Basic](/{{ page.collection }}/img/Task-Management-in-Microsoft-Outlook-with-Visual-Basic-001.png)
*Tasks in Microsoft Outlook, WWP Task Report Button (Upper Right)*

WWP Task Report for Outlook (Visual Basic for Applications Code):

{% highlight vb %}
'WWP Task Report for Outlook
'Version 0.2
'Coded by Andrew King
'
'2013-05-29 Version 0.2
'Added Lean Categorization Warnings
'Lean Category Case Sensitivity
'Code Cleanup
'
'2013-05-24 Version 0.1
'Hello World


Public Sub WWPTaskReport()
    Dim MsgBoxTitle As String
    MsgBoxTitle = "WWP Task Report for Outlook"
    
    Dim objOutlook As Outlook.Application
    Dim objSelection As Outlook.Selection
    Dim objItem As Object
        
    Dim PlannedWork As Long
    Dim ActualWork As Long
    Dim ImprovisedWork As Long
    PlannedWork = 0
    ActualWork = 0
    ImprovisedWork = 0
            
    Dim AnticipatedTotal As Integer
    Dim PlannedTotal As Integer
    Dim ImprovisedTotal As Integer
    Dim CompletedTotal As Integer
    Dim PromisedTotal As Integer
    AnticipatedTotal = 0
    PlannedTotal = 0
    ImprovisedTotal = 0
    CompletedTotal = 0
    PromisedTotal = 0
    
    Dim LeanCategoryCount As Integer
    LeanCategoryCount = 0
    
    Dim UncategorizedError As String
    Dim MultipleCategoriesError As String
    UncategorizedError = ""
    MultipleCategoriesError = ""

    On Error Resume Next

    Set objOutlook = CreateObject("Outlook.Application")
    Set objSelection = objOutlook.ActiveExplorer.Selection
    
    If objSelection.Count > 0 Then
        For Each objItem In objSelection
            If objItem.Class = olTask Then
            
                'Total Planned Work and Actual Work
                PlannedWork = PlannedWork + objItem.TotalWork
                ActualWork = ActualWork + objItem.ActualWork

                'Count Anticipated Tasks
                If LCase(objItem.Categories) Like "*anticipated*" Then
                    AnticipatedTotal = AnticipatedTotal + 1
                    LeanCategoryCount = LeanCategoryCount + 1
                End If
            
                'Count Planned Tasks
                If LCase(objItem.Categories) Like "*planned*" Then
                    PlannedTotal = PlannedTotal + 1
                    LeanCategoryCount = LeanCategoryCount + 1
                End If
            
                'Count Improvised Tasks, Total Improvised Work
                If LCase(objItem.Categories) Like "*improvised*" Then
                    ImprovisedTotal = ImprovisedTotal + 1
                    ImprovisedWork = ImprovisedWork + objItem.ActualWork
                    LeanCategoryCount = LeanCategoryCount + 1
                End If
            
                'Count Completed Tasks
                If objItem.Complete = True Then
                    CompletedTotal = CompletedTotal + 1
                End If
                
                'Count Promised Tasks
                PromisedTotal = objSelection.Count
            
                'Lean Categorization Warnings
                If LeanCategoryCount = 0 Then
                    UncategorizedError = vbNewLine & vbNewLine & vbNewLine & vbNewLine & "Warning: Lean category not assigned for one or more tasks. Task totals may be inaccurate."
                End If
                
                If LeanCategoryCount > 1 Then
                    MultipleCategoriesError = vbNewLine & vbNewLine & vbNewLine & vbNewLine & "Warning: Multiple lean categories assigned to a single task. Task totals may be inaccurate."
                End If
                
                LeanCategoryCount = 0
            
            Else
                Result = MsgBox("Incorrect selection. Only tasks are supported.", vbCritical, MsgBoxTitle)
                Exit Sub
            End If
        Next
    Else
        Result = MsgBox("No tasks selected. Please make a selection first.", vbCritical, MsgBoxTitle)
        Exit Sub
    End If

    Result = MsgBox("Planned Work: " & vbNewLine & HoursMinsMsg(PlannedWork) & vbNewLine & vbNewLine & "Actual Work: " & vbNewLine & HoursMinsMsg(ActualWork) & vbNewLine & vbNewLine & "Improvised Work: " & vbNewLine & HoursMinsMsg(ImprovisedWork) & vbNewLine & vbNewLine & vbNewLine & vbNewLine & "Anticipated Tasks: " & AnticipatedTotal & vbNewLine & vbNewLine & "Completed Tasks: " & CompletedTotal & vbNewLine & vbNewLine & "Promised Tasks: " & PromisedTotal & vbNewLine & vbNewLine & "Improvised Tasks: " & ImprovisedTotal & UncategorizedError & MultipleCategoriesError, vbInformation, MsgBoxTitle)

    Set objItem = Nothing
    Set objSelection = Nothing
    Set objOutlook = Nothing
End Sub

Public Function HoursMinsMsg(TotalMinutes As Long) As String
    Dim Hours As Long
    Dim Minutes As Long
    Hours = TotalMinutes \ 60
    Minutes = TotalMinutes Mod 60
    HoursMinsMsg = Hours & " hours and " & Minutes & " minutes"
End Function
{% endhighlight %}

![Task Management in Microsoft Outlook with Visual Basic](/{{ page.collection }}/img/Task-Management-in-Microsoft-Outlook-with-Visual-Basic-002.png)
