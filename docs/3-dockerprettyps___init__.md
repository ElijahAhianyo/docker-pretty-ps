# Dockerprettyps Init

[_Documentation generated by Documatic_](https://www.documatic.com)

<!---Documatic-section-Codebase Structure-start--->
## Codebase Structure

<!---Documatic-block-system_architecture-start--->
```mermaid
None
```
<!---Documatic-block-system_architecture-end--->

# #
<!---Documatic-section-Codebase Structure-end--->

<!---Documatic-section-dockerprettyps.__init__.run_cli-start--->
## dockerprettyps.__init__.run_cli

<!---Documatic-section-run_cli-start--->
<!---Documatic-block-dockerprettyps.__init__.run_cli-start--->
<details>
	<summary><code>dockerprettyps.__init__.run_cli</code> code snippet</summary>

```python
def run_cli():
    args = _parsed_args()
    if args.version:
        version()
        exit()
    try:
        raw_containers = get_raw_containers()
    except errors.BadResponseDockerEngine:
        print('%sError:%s Bad response from the Docker Engine' % (RED, ENDC))
        exit(1)
    containers = clean_output(raw_containers)
    total_containers = len(containers)
    total_running_containers = _get_num_running_containers(containers)
    containers = filter_containers(containers, args)
    containers = order_containers(containers, args)
    if args.json:
        give_json(containers, args)
    else:
        print_format(containers, total_containers, total_running_containers, args)
```
</details>
<!---Documatic-block-dockerprettyps.__init__.run_cli-end--->
<!---Documatic-section-run_cli-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.run_cli-end--->

<!---Documatic-section-dockerprettyps.__init__._parsed_args-start--->
## dockerprettyps.__init__._parsed_args

<!---Documatic-section-_parsed_args-start--->
<!---Documatic-block-dockerprettyps.__init__._parsed_args-start--->
<details>
	<summary><code>dockerprettyps.__init__._parsed_args</code> code snippet</summary>

```python
def _parsed_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('search', nargs='?', default='', help='Phrase to search containers, comma separate multiples.')
    parser.add_argument('-a', '--all', default=False, action='store_true', help='Selects against all running and stopped containers')
    parser.add_argument('-s', '--slim', default=False, action='store_true', help='Shows a slim minimal output.')
    parser.add_argument('-i', '--include', default=[], help='Data points to add to display, (c)reated, (p)orts, (i)mage_id, co(m)mand')
    parser.add_argument('-o', '--order', nargs='?', default='', help="Order by, defaults to container start, allows 'container', 'pretty_print_fmt_containers'.")
    parser.add_argument('-r', '--reverse', default=False, action='store_true', help='Reverses the display order.')
    parser.add_argument('-j', '--json', default='', action='store_true', help='Instead of printing, creates a json response of the container data.')
    parser.add_argument('-v', '--version', default=False, action='store_true', help='Print the binary version information.')
    args = parser.parse_args()
    includes = []
    if args.include:
        for letter in args.include:
            includes.append(letter)
        args.include = includes
    if ',' in args.search:
        searches = args.search.split(',')
    elif args.search == '':
        searches = []
    else:
        searches = [args.search]
    args.search = searches
    return args
```
</details>
<!---Documatic-block-dockerprettyps.__init__._parsed_args-end--->
<!---Documatic-section-_parsed_args-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._parsed_args-end--->

<!---Documatic-section-dockerprettyps.__init__.version-start--->
## dockerprettyps.__init__.version

<!---Documatic-section-version-start--->
<!---Documatic-block-dockerprettyps.__init__.version-start--->
<details>
	<summary><code>dockerprettyps.__init__.version</code> code snippet</summary>

```python
def version():
    spacing = '                                                        '
    print(__title__)
    print('\t%sdocker-pretty-ps%s                                Version: %s' % (BOLD, ENDC, __version__))
    print('%s@politeauthority' % spacing)
    print('%shttps://github.com/politeauthority/docker-pretty-ps\n\n' % spacing)
    return True
```
</details>
<!---Documatic-block-dockerprettyps.__init__.version-end--->
<!---Documatic-section-version-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.version-end--->

<!---Documatic-section-dockerprettyps.__init__.get_raw_containers-start--->
## dockerprettyps.__init__.get_raw_containers

<!---Documatic-section-get_raw_containers-start--->
<!---Documatic-block-dockerprettyps.__init__.get_raw_containers-start--->
<details>
	<summary><code>dockerprettyps.__init__.get_raw_containers</code> code snippet</summary>

```python
def get_raw_containers():
    cmds = ['docker', 'ps', '-a']
    out = subprocess.Popen(cmds, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    (stdout, stderr) = out.communicate()
    stdout = stdout.decode('utf-8')
    if 'Error' in stdout or 'Cannot connect' in stdout:
        raise errors.BadResponseDockerEngine
    return stdout
```
</details>
<!---Documatic-block-dockerprettyps.__init__.get_raw_containers-end--->
<!---Documatic-section-get_raw_containers-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.get_raw_containers-end--->

<!---Documatic-section-dockerprettyps.__init__.clean_output-start--->
## dockerprettyps.__init__.clean_output

<!---Documatic-section-clean_output-start--->
<!---Documatic-block-dockerprettyps.__init__.clean_output-start--->
<details>
	<summary><code>dockerprettyps.__init__.clean_output</code> code snippet</summary>

```python
def clean_output(output):
    lines = output.split('\n')
    containers = []
    for line in lines[1:]:
        line_split = line.split('  ')
        revised_line_split = []
        if len(line_split) == 1:
            continue
        for piece in line_split:
            if piece and piece.strip() != '':
                revised_line_split.append(piece)
        container = {'container_id': revised_line_split[0].strip(), 'image': revised_line_split[1].strip(), 'command': revised_line_split[2].strip().replace('"', ''), 'created': revised_line_split[3].strip(), 'created_date': _parse_ps_date(revised_line_split[4].strip()), 'status': revised_line_split[4].strip(), 'status_date': _parse_ps_date(revised_line_split[4].strip()), 'running': _clean_status(revised_line_split[4])}
        if len(revised_line_split) == 6:
            container['ports'] = []
            container['name'] = revised_line_split[5].strip()
        else:
            container['ports'] = _parse_ports(revised_line_split[5])
            container['name'] = revised_line_split[6].strip()
        containers.append(container)
    containers = get_container_colors(containers)
    return containers
```
</details>
<!---Documatic-block-dockerprettyps.__init__.clean_output-end--->
<!---Documatic-section-clean_output-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.clean_output-end--->

<!---Documatic-section-dockerprettyps.__init__._parse_ports-start--->
## dockerprettyps.__init__._parse_ports

<!---Documatic-section-_parse_ports-start--->
<!---Documatic-block-dockerprettyps.__init__._parse_ports-start--->
<details>
	<summary><code>dockerprettyps.__init__._parse_ports</code> code snippet</summary>

```python
def _parse_ports(port_str):
    port_str = port_str.strip()
    if port_str == '':
        return []
    elif ', ' not in port_str:
        return [port_str]
    ports = port_str.split(', ')
    return ports
```
</details>
<!---Documatic-block-dockerprettyps.__init__._parse_ports-end--->
<!---Documatic-section-_parse_ports-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._parse_ports-end--->

<!---Documatic-section-dockerprettyps.__init__._parse_ps_date-start--->
## dockerprettyps.__init__._parse_ps_date

<!---Documatic-section-_parse_ps_date-start--->
<!---Documatic-block-dockerprettyps.__init__._parse_ps_date-start--->
<details>
	<summary><code>dockerprettyps.__init__._parse_ps_date</code> code snippet</summary>

```python
def _parse_ps_date(val):
    now = datetime.now()
    cleaned = val.replace('Up ', '').lower()
    if 'restarting' in cleaned:
        cleaned = cleaned[cleaned.find(')') + 1:]
    cleaned = cleaned.replace('ago', '').strip()
    if '(healthy' in cleaned:
        cleaned = cleaned[:cleaned.find('(healthy')].strip()
    elif '(health' in cleaned:
        cleaned = cleaned[:cleaned.find(')') + 1].strip()
    elif '(unhealthy' in cleaned:
        cleaned = cleaned[:cleaned.find('(')].strip()
    elif 'exited (' in cleaned:
        cleaned = cleaned[cleaned.find(')') + 1:].strip()
    if 'seconds' in cleaned:
        digit = cleaned.replace(' seconds', '')
        digit = int(digit)
        the_date = now - timedelta(seconds=digit)
    elif 'minutes' in cleaned:
        digit = cleaned.replace(' minutes', '')
        digit = int(digit)
        the_date = now - timedelta(minutes=digit)
    elif 'about an hour' in cleaned:
        digit = 1
        the_date = now - timedelta(hours=digit)
    elif 'hours' in cleaned:
        digit = cleaned.replace(' hours', '')
        digit = int(digit)
        the_date = now - timedelta(hours=digit)
    elif 'days' in cleaned:
        digit = cleaned.replace(' days', '')
        digit = int(digit)
        the_date = now - timedelta(days=digit)
    else:
        the_date = now
    return the_date
```
</details>
<!---Documatic-block-dockerprettyps.__init__._parse_ps_date-end--->
<!---Documatic-section-_parse_ps_date-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._parse_ps_date-end--->

<!---Documatic-section-dockerprettyps.__init__._clean_status-start--->
## dockerprettyps.__init__._clean_status

<!---Documatic-section-_clean_status-start--->
<!---Documatic-block-dockerprettyps.__init__._clean_status-start--->
<details>
	<summary><code>dockerprettyps.__init__._clean_status</code> code snippet</summary>

```python
def _clean_status(val):
    val = val.lower().strip()
    if 'exited (' in val or 'created' == val:
        return False
    return True
```
</details>
<!---Documatic-block-dockerprettyps.__init__._clean_status-end--->
<!---Documatic-section-_clean_status-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._clean_status-end--->

<!---Documatic-section-dockerprettyps.__init__.get_container_colors-start--->
## dockerprettyps.__init__.get_container_colors

<!---Documatic-section-get_container_colors-start--->
<!---Documatic-block-dockerprettyps.__init__.get_container_colors-start--->
<details>
	<summary><code>dockerprettyps.__init__.get_container_colors</code> code snippet</summary>

```python
def get_container_colors(containers):
    count = 0
    for c in containers:
        c['color'] = get_color(count)
        count += 1
    return containers
```
</details>
<!---Documatic-block-dockerprettyps.__init__.get_container_colors-end--->
<!---Documatic-section-get_container_colors-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.get_container_colors-end--->

<!---Documatic-section-dockerprettyps.__init__.get_color-start--->
## dockerprettyps.__init__.get_color

<!---Documatic-section-get_color-start--->
<!---Documatic-block-dockerprettyps.__init__.get_color-start--->
<details>
	<summary><code>dockerprettyps.__init__.get_color</code> code snippet</summary>

```python
def get_color(count):
    colors = ['\x1b[94m', GREEN, RED, '\x1b[96m', '\x1b[93m', '\x1b[95m']
    while count > len(colors):
        colors += colors
    return colors[count - 1]
```
</details>
<!---Documatic-block-dockerprettyps.__init__.get_color-end--->
<!---Documatic-section-get_color-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.get_color-end--->

<!---Documatic-section-dockerprettyps.__init__._get_num_running_containers-start--->
## dockerprettyps.__init__._get_num_running_containers

<!---Documatic-section-_get_num_running_containers-start--->
<!---Documatic-block-dockerprettyps.__init__._get_num_running_containers-start--->
<details>
	<summary><code>dockerprettyps.__init__._get_num_running_containers</code> code snippet</summary>

```python
def _get_num_running_containers(containers):
    total_running = 0
    for container in containers:
        if container['running']:
            total_running += 1
    return total_running
```
</details>
<!---Documatic-block-dockerprettyps.__init__._get_num_running_containers-end--->
<!---Documatic-section-_get_num_running_containers-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._get_num_running_containers-end--->

<!---Documatic-section-dockerprettyps.__init__.filter_containers-start--->
## dockerprettyps.__init__.filter_containers

<!---Documatic-section-filter_containers-start--->
<!---Documatic-block-dockerprettyps.__init__.filter_containers-start--->
<details>
	<summary><code>dockerprettyps.__init__.filter_containers</code> code snippet</summary>

```python
def filter_containers(containers, args):
    if not args.search and args.all:
        return containers
    filtered_containers = []
    if args.search:
        for container in containers:
            for search in args.search:
                if search in container['name']:
                    filtered_containers.append(container)
                    break
    else:
        filtered_containers = containers
    more_filtered_containers = []
    if not args.all:
        for container in filtered_containers:
            if container['running']:
                more_filtered_containers.append(container)
    else:
        more_filtered_containers = filtered_containers
    return more_filtered_containers
```
</details>
<!---Documatic-block-dockerprettyps.__init__.filter_containers-end--->
<!---Documatic-section-filter_containers-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.filter_containers-end--->

<!---Documatic-section-dockerprettyps.__init__.order_containers-start--->
## dockerprettyps.__init__.order_containers

<!---Documatic-section-order_containers-start--->
<!---Documatic-block-dockerprettyps.__init__.order_containers-start--->
<details>
	<summary><code>dockerprettyps.__init__.order_containers</code> code snippet</summary>

```python
def order_containers(containers, args):
    if not containers:
        return containers
    field = 'status_date'
    if args.order:
        if args.order in ['name', 'container']:
            field = 'name'
        if args.order in ['created', 'create']:
            field = 'created_date'
    ordered_containers = sorted(containers, key=itemgetter(field))
    if args.reverse:
        ordered_containers.reverse()
    return ordered_containers
```
</details>
<!---Documatic-block-dockerprettyps.__init__.order_containers-end--->
<!---Documatic-section-order_containers-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.order_containers-end--->

<!---Documatic-section-dockerprettyps.__init__.print_format-start--->
## dockerprettyps.__init__.print_format

<!---Documatic-section-print_format-start--->
<!---Documatic-block-dockerprettyps.__init__.print_format-start--->
<details>
	<summary><code>dockerprettyps.__init__.print_format</code> code snippet</summary>

```python
def print_format(containers, total_containers, total_running_containers, args):
    if args.search:
        if not args.all:
            print('Currently running containers with: "%s" \n' % '", "'.join(args.search))
        else:
            print('All cotnainers containers with: "%s" \n' % '", "'.join(args.search))
    elif not args.all:
        print('All currently running docker containers\n')
    else:
        print('All docker containers\n')
    pretty_print_fmt_containers(containers, args)
    print('\nTotal containers:\t%s' % total_containers)
    print('Total running:\t\t%s' % total_running_containers)
    if args.search:
        print('Containers in search:\t%s' % len(containers))
    return True
```
</details>
<!---Documatic-block-dockerprettyps.__init__.print_format-end--->
<!---Documatic-section-print_format-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.print_format-end--->

<!---Documatic-section-dockerprettyps.__init__.pretty_print_fmt_containers-start--->
## dockerprettyps.__init__.pretty_print_fmt_containers

<!---Documatic-section-pretty_print_fmt_containers-start--->
<!---Documatic-block-dockerprettyps.__init__.pretty_print_fmt_containers-start--->
<details>
	<summary><code>dockerprettyps.__init__.pretty_print_fmt_containers</code> code snippet</summary>

```python
def pretty_print_fmt_containers(containers, args):
    selected_includes = ['r', 's', 'c', 'p', 'n', 'i', 'm']
    if args.slim or args.include:
        selected_includes = []
    if args.include:
        for include in args.include:
            selected_includes.append(include)
    print_content = {}
    for container in containers:
        container_content = {'display_name': container_display_name(container, args), 'data': []}
        if 'n' in selected_includes:
            container_content['data'].append([BOLD + '\tContainer ID:' + ENDC, container['container_id']])
        if 'i' in selected_includes:
            container_content['data'].append([BOLD + '\tImage:' + ENDC, container['image']])
        if 'm' in selected_includes:
            container_content['data'].append([BOLD + '\tCommand:' + ENDC, container['command']])
        created_data = _handle_column_created(args, container, selected_includes)
        if created_data:
            container_content['data'] += created_data
        status_data = _handle_column_status(container, selected_includes, args)
        if status_data:
            container_content['data'] += status_data
        state_data = _handle_column_state(container, selected_includes, args)
        if state_data:
            container_content['data'] += state_data
        ports_data = _handle_column_ports(args, container, selected_includes)
        if ports_data:
            container_content['data'] += ports_data
        print_content[container['name']] = container_content
    print_data(print_content)
    return True
```
</details>
<!---Documatic-block-dockerprettyps.__init__.pretty_print_fmt_containers-end--->
<!---Documatic-section-pretty_print_fmt_containers-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.pretty_print_fmt_containers-end--->

<!---Documatic-section-dockerprettyps.__init__.container_display_name-start--->
## dockerprettyps.__init__.container_display_name

<!---Documatic-section-container_display_name-start--->
<!---Documatic-block-dockerprettyps.__init__.container_display_name-start--->
<details>
	<summary><code>dockerprettyps.__init__.container_display_name</code> code snippet</summary>

```python
def container_display_name(container, args):
    if not args.search:
        return container['color'] + container['name'] + ENDC
    else:
        highlighted_name = container['color'] + container['name']
        for search in args.search:
            highlighted_name = highlighted_name.replace(search, BOLD + container['color'] + search + ENDC + container['color'])
        return highlighted_name + ENDC
```
</details>
<!---Documatic-block-dockerprettyps.__init__.container_display_name-end--->
<!---Documatic-section-container_display_name-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.container_display_name-end--->

<!---Documatic-section-dockerprettyps.__init__._handle_column_state-start--->
## dockerprettyps.__init__._handle_column_state

<!---Documatic-section-_handle_column_state-start--->
<!---Documatic-block-dockerprettyps.__init__._handle_column_state-start--->
<details>
	<summary><code>dockerprettyps.__init__._handle_column_state</code> code snippet</summary>

```python
def _handle_column_state(container, selected_includes, args):
    print_d = []
    if args.all and 'r' in selected_includes:
        if container['running']:
            print_d.append([BOLD + '\tState:' + ENDC, GREEN + '[ON]' + ENDC])
        else:
            print_d.append([BOLD + '\tState:' + ENDC, RED + '[OFF]' + ENDC])
    return print_d
```
</details>
<!---Documatic-block-dockerprettyps.__init__._handle_column_state-end--->
<!---Documatic-section-_handle_column_state-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._handle_column_state-end--->

<!---Documatic-section-dockerprettyps.__init__._handle_column_status-start--->
## dockerprettyps.__init__._handle_column_status

<!---Documatic-section-_handle_column_status-start--->
<!---Documatic-block-dockerprettyps.__init__._handle_column_status-start--->
<details>
	<summary><code>dockerprettyps.__init__._handle_column_status</code> code snippet</summary>

```python
def _handle_column_status(container, selected_includes, args):
    print_d = []
    if 's' in selected_includes:
        print_d.append([BOLD + '\tStatus:' + ENDC, container['status']])
    return print_d
```
</details>
<!---Documatic-block-dockerprettyps.__init__._handle_column_status-end--->
<!---Documatic-section-_handle_column_status-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._handle_column_status-end--->

<!---Documatic-section-dockerprettyps.__init__._handle_column_ports-start--->
## dockerprettyps.__init__._handle_column_ports

<!---Documatic-section-_handle_column_ports-start--->
<!---Documatic-block-dockerprettyps.__init__._handle_column_ports-start--->
<details>
	<summary><code>dockerprettyps.__init__._handle_column_ports</code> code snippet</summary>

```python
def _handle_column_ports(args, container, selected_includes):
    print_d = []
    if 'p' in selected_includes:
        if len(container['ports']) == 0:
            print_d.append([BOLD + '\tPorts:' + ENDC, ''])
        elif len(container['ports']) == 1:
            print_d.append([BOLD + '\tPorts:' + ENDC, container['ports'][0]])
        else:
            c = 0
            for container_port in container['ports']:
                container_port = container_port.strip()
                if c == 0:
                    print_d.append([BOLD + '\tPorts:' + ENDC, container_port])
                else:
                    print_d.append(['', container_port])
                c += 1
    return print_d
```
</details>
<!---Documatic-block-dockerprettyps.__init__._handle_column_ports-end--->
<!---Documatic-section-_handle_column_ports-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._handle_column_ports-end--->

<!---Documatic-section-dockerprettyps.__init__._handle_column_created-start--->
## dockerprettyps.__init__._handle_column_created

<!---Documatic-section-_handle_column_created-start--->
<!---Documatic-block-dockerprettyps.__init__._handle_column_created-start--->
<details>
	<summary><code>dockerprettyps.__init__._handle_column_created</code> code snippet</summary>

```python
def _handle_column_created(args, container, selected_includes):
    print_d = []
    if 'c' in selected_includes:
        print_d.append([BOLD + '\tCreated:' + ENDC, container['created']])
    return print_d
```
</details>
<!---Documatic-block-dockerprettyps.__init__._handle_column_created-end--->
<!---Documatic-section-_handle_column_created-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._handle_column_created-end--->

<!---Documatic-section-dockerprettyps.__init__.print_data-start--->
## dockerprettyps.__init__.print_data

<!---Documatic-section-print_data-start--->
<!---Documatic-block-dockerprettyps.__init__.print_data-start--->
<details>
	<summary><code>dockerprettyps.__init__.print_data</code> code snippet</summary>

```python
def print_data(container_info):
    for (container_name, container) in container_info.items():
        print(container['display_name'])
        if not container['data']:
            continue
        col_width = 30
        for row in container['data']:
            if len(row[0]) == 0:
                print('                              %s' % row[1])
            else:
                print('%s %s' % (row[0].ljust(col_width), row[1]))
        print('')
```
</details>
<!---Documatic-block-dockerprettyps.__init__.print_data-end--->
<!---Documatic-section-print_data-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.print_data-end--->

<!---Documatic-section-dockerprettyps.__init__.give_json-start--->
## dockerprettyps.__init__.give_json

<!---Documatic-section-give_json-start--->
<!---Documatic-block-dockerprettyps.__init__.give_json-start--->
<details>
	<summary><code>dockerprettyps.__init__.give_json</code> code snippet</summary>

```python
def give_json(containers, args):
    clean_date_containers = _json_container_dates(containers)
    more_cleaned = []
    for container in clean_date_containers:
        more_cleaned.append(container.pop('color'))
    ret_dict = {'total_containers': len(containers), 'containers': clean_date_containers}
    print(json.dumps(ret_dict, indent=4, sort_keys=True))
    return True
```
</details>
<!---Documatic-block-dockerprettyps.__init__.give_json-end--->
<!---Documatic-section-give_json-end--->

# #
<!---Documatic-section-dockerprettyps.__init__.give_json-end--->

<!---Documatic-section-dockerprettyps.__init__._json_container_dates-start--->
## dockerprettyps.__init__._json_container_dates

<!---Documatic-section-_json_container_dates-start--->
<!---Documatic-block-dockerprettyps.__init__._json_container_dates-start--->
<details>
	<summary><code>dockerprettyps.__init__._json_container_dates</code> code snippet</summary>

```python
def _json_container_dates(containers):
    clean_containers = []
    for container in containers:
        tmp_container = container
        tmp_container['status_date'] = str(tmp_container['status_date'])
        tmp_container['created_date'] = str(tmp_container['created_date'])
        clean_containers.append(tmp_container)
    return clean_containers
```
</details>
<!---Documatic-block-dockerprettyps.__init__._json_container_dates-end--->
<!---Documatic-section-_json_container_dates-end--->

# #
<!---Documatic-section-dockerprettyps.__init__._json_container_dates-end--->

[_Documentation generated by Documatic_](https://www.documatic.com)