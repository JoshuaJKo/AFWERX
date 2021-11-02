from numpy.lib.function_base import average
import pandas as pd
import datetime
import csv


def create_dateTime(date: str):
    if(date == 'nan'):
        return date

    month, date = find_and_cut(date, '/')
    day, date = find_and_cut(date, '/')
    year, date = find_and_cut(date, ' ')
    hour, date = find_and_cut(date, ':')
    minute, date = find_and_cut(date, ':')
    second, date = find_and_cut(date, ' ')

    if date == 'PM' and hour != 12:
        hour += 12

    # datetime(year, month, day, hour, minute ,second)
    return datetime.datetime(year, month, day, hour, minute, second)


def find_and_cut(to_cut: str, cut_at_char: str):

    index = to_cut.find(cut_at_char)
    cut_value = to_cut[0:index]
    to_cut = to_cut[index+1:]

    return int(cut_value), str(to_cut)


ARPCMetrics = pd.read_csv('ARPCmetrics15Aug-09Oct.csv')

all_deltas = list()
assigned_support_list = dict()

num_resolved = 0
num_unresolved = 0
total_time = datetime.datetime(1, 1, 1, 1, 1, 1)

for index, row in ARPCMetrics.iterrows():
    submit_time = create_dateTime(str(row['TICKET_SUBMIT_DATE']))
    resolved_time = create_dateTime(str(row['LAST_RESOLVED_DATE']))

    if(str(row['ASSIGNED_SUPPORT_ORGANIZATION']) not in assigned_support_list):
        assigned_support_list[(
            str(row['ASSIGNED_SUPPORT_ORGANIZATION']))] = list()

    if(resolved_time == 'nan'):
        num_unresolved += 1
    else:
        delta = resolved_time - submit_time
        total_time += delta
        num_resolved += 1
        all_deltas.append(delta)

        assigned_support_list[(
            str(row['ASSIGNED_SUPPORT_ORGANIZATION']))].append(delta)

total_time -= datetime.datetime(1, 1, 1, 1, 1, 1)
average_time = (total_time/num_resolved)
rows = []

for org in assigned_support_list:
    total_time = datetime.datetime(1, 1, 1, 1, 1, 1)
    for time in assigned_support_list[org]:
        total_time += time
    total_time -= datetime.datetime(1, 1, 1, 1, 1, 1)

    if total_time.days != 0:
        avg_assigned_time = total_time/len(assigned_support_list[org])
    print(org, total_time, avg_assigned_time)
    rows.append([str(org), str(total_time), str(avg_assigned_time)])

print(rows)

fields = ['Assigned_Org_Name', 'Total_time', 'Average_time']
filename = "metrics.csv"

with open(filename, 'w') as csvfile:
    csvwriter = csv.writer(csvfile)
    csvwriter.writerow(fields)

    for row in rows:
        csvwriter.writerow(row)

csvfile.close()
