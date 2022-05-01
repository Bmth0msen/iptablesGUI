# iptablesGUI
A simple GUI for the Linux based firewall iptables



#!/usr/bin/env python3
import PySimpleGUI as sg
import subprocess

sg.theme('DarkAmber')   # Add a touch of color
# All the stuff inside your window.
layout = [[sg.Column([[sg.Text(text="Source IP: ")],[sg.Text(text="Destination IP: ")]]), sg.Column([[sg.InputText(key="-srcip-", size=(16,1))],[sg.InputText(key="-dstip-", size=(16,1))]]), sg.Column([[sg.Text(text="Source Port: ")],[sg.Text(text="Destination Port: ")]]), sg.Column([[sg.InputText(key="-srcpt-", size=(6,1))],[sg.InputText(key="-dstpt-", size=(6,1))]]), sg.Column([[sg.Frame(None, [[sg.Radio("TCP", "proto", default=True, key="-TCP-"), sg.Radio("UDP", "proto", key="-UDP-")]])], [sg.Frame(None, [[sg.Radio("ACCEPT", "rlact", default=True, key="-ACCEPT-"), sg.Radio("DROP", "rlact", key="-DROP-"), sg.Radio("REJECT", "rlact", key="-REJECT-")]])]])], [sg.Multiline(default_text="Click 'Check Rules' button to show current iptables rules here", size=(50,15), key="-OUTBOX-"), sg.Column([[sg.Button('Check Rules', size=41)],[sg.Button('Delete inbound rule line: ', size=35), sg.InputText(key="-inruleline-", size=(3,1), default_text="1")], [sg.Button('Delete outbound rule line: ', size=35), sg.InputText(key="-outruleline-", size=(3,1), default_text="1")], [sg.Button('FLUSH ALL RULES', size=41)], [sg.Button('Add Inbound Rule', size=18), sg.Button('Add Outbound Rule', size=18)], [sg.Button('Save', size=18), sg.Button('Exit', size=18)]])]]

# Create the Window
window = sg.Window('IP Tables GUI Project', layout)
# Event Loop to process "events" and get the "values" of the inputs
while True:
  event, values = window.read()
  if event == sg.WIN_CLOSED or event == 'Exit': # if user closes window or clicks cancel
    break

  if event == 'Check Rules':
    table = subprocess.run(['iptables', '-L'], capture_output=True, text=True)
    window["-OUTBOX-"].update(table.stdout)
    print(table.stdout)
  srcip = values["-srcip-"]
  srcpt = values["-srcpt-"]
  dstip = values["-dstip-"]
  dstpt = values["-dstpt-"]
  if values["-TCP-"] == True:
    proto = "TCP"
  else:
    proto = "UDP"
  if values["-ACCEPT-"] == True:
    rlact = "ACCEPT"
  elif values["-DROP-"] == True:
    rlact = "DROP"
  else:
    rlact = "REJECT"

  if event == 'Add Inbound Rule':
    subprocess.run(['iptables', '-A', 'INPUT', '-p', proto, '-s', srcip, '--sport', srcpt, '-d', dstip, '--dport', dstpt, '-j', rlact])
  if event == 'Add Outbound Rule':
    subprocess.run(['iptables', '-A', 'OUTPUT', '-p', proto, '-s', srcip, '--sport', srcpt, '-d', dstip, '--dport', dstpt, '-j', rlact])

  if event == 'Delete inbound rule line: ':
    inruleline = int(values["-inruleline-"])
    subprocess.run(['iptables', '-D', 'INPUT', inruleline])
  if event == 'Delete outbound rule line: ':
    outruleline = int(values["-outruleline-"])
    subprocess.run(['iptables', '-D', 'OUTPUT', outruleline])
  if event == 'FLUSH ALL RULES':
    subprocess.run(['iptables', '-F'])

  if event == 'Save':
    subprocess.run(['iptables-save'])
window.close()
