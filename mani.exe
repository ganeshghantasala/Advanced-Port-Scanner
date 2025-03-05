import os
import socket
import threading
import tkinter as tk
from tkinter import scrolledtext
from datetime import datetime
from pathlib import Path
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Function to resolve domain to IP
def resolve_domain(host):
    try:
        return socket.gethostbyname(host)
    except socket.gaierror:
        return None

# Function to scan a TCP port
def scan_tcp(host, port, output_widget):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(0.5)
        result = s.connect_ex((host, port))
        if result == 0:
            output_widget.insert(tk.END, f"TCP Port {port}: Open\n")
            output_widget.see(tk.END)
            return {'Port': port, 'Protocol': 'TCP', 'Status': 'Open'}
        s.close()
    except:
        pass
    return None

# Function to scan a UDP port
def scan_udp(host, port, output_widget):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.settimeout(0.5)
        s.sendto(b'', (host, port))
        s.recvfrom(1024)
        output_widget.insert(tk.END, f"UDP Port {port}: Open\n")
        output_widget.see(tk.END)
        return {'Port': port, 'Protocol': 'UDP', 'Status': 'Open'}
    except:
        pass
    return None

# Function to scan ports
def scan_ports(host, protocol, start_port, end_port, specific_ports, output_widget):
    open_ports = []
    ports_to_scan = (
        [int(port) for port in specific_ports.split(',')]
        if specific_ports
        else range(start_port, end_port + 1)
    )

    for port in ports_to_scan:
        if protocol == 'TCP':
            result = scan_tcp(host, port, output_widget)
        elif protocol == 'UDP':
            result = scan_udp(host, port, output_widget)
        elif protocol == 'Both':
            result = scan_tcp(host, port, output_widget) or scan_udp(
                host, port, output_widget
            )
        if result:
            open_ports.append(result)

    return open_ports

# Function to generate and save report
def generate_report(host, open_ports):
    timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    downloads_folder = Path.home() / "Downloads"
    downloads_folder.mkdir(exist_ok=True)
    filename = downloads_folder / f"port_scan_report_{host}_{timestamp}.txt"

    with open(filename, 'w') as f:
        f.write(f"Port Scan Report\n")
        f.write(f"Host: {host}\n")
        f.write(f"Scan Time: {timestamp}\n\n")
        f.write(f"{'Port':<10}{'Protocol':<10}{'Status':<10}\n")
        f.write("=" * 40 + "\n")
        for entry in open_ports:
            f.write(
                f"{entry['Port']:<10}{entry['Protocol']:<10}{entry['Status']:<10}\n"
            )
    return filename

# Function to send email report
def send_email_report(filename, recipient_email, output_widget):
    sender_email = "lingamalluvasu123@gmail.com"  
    sender_password = "mtqmvucukcdcivyf"  
    subject = "Port Scan Report"

    try:
        with open(filename, 'r') as f:
            report_content = f.read()

        msg = MIMEMultipart()
        msg['From'] = sender_email
        msg['To'] = recipient_email
        msg['Subject'] = subject
        msg.attach(MIMEText(report_content, 'plain'))

        with smtplib.SMTP('smtp.gmail.com', 587) as server:
            server.starttls()
            server.login(sender_email, sender_password)
            server.sendmail(sender_email, recipient_email, msg.as_string())

        output_widget.insert(tk.END, "Report sent successfully.\n")
    except Exception as e:
        output_widget.insert(tk.END, f"Error sending email: {e}\n")

# Function to start scanning
def start_scan():
    host = entry_host.get()
    start_port = int(entry_start_port.get())
    end_port = int(entry_end_port.get())
    protocol = selected_protocol.get()
    specific_ports = entry_specific_ports.get()
    recipient_email = entry_email.get()

    output_widget.delete(1.0, tk.END)
    ip = resolve_domain(host)
    if not ip:
        output_widget.insert(tk.END, f"Unable to resolve domain: {host}\n")
        return

    output_widget.insert(tk.END, f"Resolved IP: {ip}\n")
    output_widget.insert(tk.END, f"Scanning ports...\n")

    open_ports = scan_ports(ip, protocol, start_port, end_port, specific_ports, output_widget)

    if open_ports:
        output_widget.insert(tk.END, f"Scan complete. Generating report...\n")
        filename = generate_report(host, open_ports)
        output_widget.insert(tk.END, f"Report saved at: {filename}\n")

        if recipient_email:
            output_widget.insert(tk.END, f"Sending report to {recipient_email}...\n")
            send_email_report(filename, recipient_email, output_widget)
    else:
        output_widget.insert(tk.END, f"No open ports found.\n")

# Tkinter setup
root = tk.Tk()
root.title("Port Scanner")

# Inputs
tk.Label(root, text="Host").grid(row=0, column=0)
entry_host = tk.Entry(root, width=30)
entry_host.grid(row=0, column=1)

tk.Label(root, text="Start Port").grid(row=1, column=0)
entry_start_port = tk.Entry(root, width=30)
entry_start_port.grid(row=1, column=1)

tk.Label(root, text="End Port").grid(row=2, column=0)
entry_end_port = tk.Entry(root, width=30)
entry_end_port.grid(row=2, column=1)

tk.Label(root, text="Specific Ports (comma separated)").grid(row=3, column=0)
entry_specific_ports = tk.Entry(root, width=30)
entry_specific_ports.grid(row=3, column=1)

tk.Label(root, text="Protocol").grid(row=4, column=0)
selected_protocol = tk.StringVar(value="TCP")
tk.Radiobutton(root, text="TCP", variable=selected_protocol, value="TCP").grid(row=4, column=1, sticky="w")
tk.Radiobutton(root, text="UDP", variable=selected_protocol, value="UDP").grid(row=4, column=2, sticky="w")
tk.Radiobutton(root, text="Both", variable=selected_protocol, value="Both").grid(row=4, column=3, sticky="w")

tk.Label(root, text="Recipient Email").grid(row=5, column=0)
entry_email = tk.Entry(root, width=30)
entry_email.grid(row=5, column=1)

# Output area
output_widget = scrolledtext.ScrolledText(root, width=70, height=20)
output_widget.grid(row=6, column=0, columnspan=4)

# Scan button
button_scan = tk.Button(root, text="Start Scan", command=start_scan)
button_scan.grid(row=7, column=1)

# Tkinter main loop
root.mainloop()
