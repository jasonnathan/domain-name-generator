Another very cool project (not mine but forked)

# META: DOMAIN-NAME-GENERATOR

## Folder Structure

```plaintext
/Users/jasonnathan/Repos/domain-name-generator
├── LICENSE
├── README.md
└── generatedomain.py

1 directory, 3 files
```
## File: README.md

```md
# Domain Name Generator (and availability checker)

A simple Python command line domain name generator which I wrote after finding quite a lot of the web-based tools lacking for my purposes, which was:

 * generate a simple 2+ word domain (based on keyword 'groups' - e.g. "smart,smarter,clever,intelligent" is a single group)
 * only look at .com.
 * don't report registered (or 'premium') domains.

This tool generates keywords based on the provided arguments and then checks if they are registered or not (and by default only outputs available ones).
 
Usage is command line:

```
$ python generatedomain.py
usage: generatedomain.py [-h] [--skip-whois] [--show-taken]
                         [--starts-with STARTS_WITH] [--ends-with ENDS_WITH]
                         --kws KWS [KWS ...]


$ python generatedomain.py --kws keyword1 keyword2 a,few,different,options
keyword1keyword2a.com is available
keyword1keyword2few.com is available
keyword1keyword2different.com is available
keyword1keyword2options.com is available


$ python generatedomain.py --show-taken --kws demo,demonstration,example,exhibit tool,software,program
demotool.com is NOT available; expiry date is 2020-09-22T18:32:37Z
demosoftware.com is NOT available; expiry date is 2020-06-11T12:43:47Z
demoprogram.com is NOT available; expiry date is 2020-01-16T20:10:59Z
demonstrationtool.com is available
demonstrationsoftware.com is NOT available; expiry date is 2020-08-02T18:16:09Z
demonstrationprogram.com is available
exampletool.com is available
examplesoftware.com is NOT available; expiry date is 2025-10-16T19:45:49Z
exampleprogram.com is NOT available; expiry date is 2021-02-15T07:21:18Z
exhibittool.com is available
exhibitsoftware.com is NOT available; expiry date is 2020-09-11T13:37:07Z
exhibitprogram.com is NOT available; expiry date is 2020-06-02T15:34:20Z

python generatedomain.py --starts-with pre --ends-with post --kws keyword1 keyword2 a,few,different,options
prekeyword1keyword2apost.com is available
prekeyword1keyword2fewpost.com is available
prekeyword1keyword2differentpost.com is available
prekeyword1keyword2optionspost.com is available
```

This tool creates all possible combinations from the provided keywords.

```

## File: generatedomain.py

```py
#!/usr/bin/env  python

import argparse
import itertools
import re
import socket


def whois_request(domain, server='whois.verisign-grs.com', port=43):
    """
    Carries out the WHOIS request for a particular domain name, against a particular registrar.
    This is not .com specific.
    """
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((server, port))
    sock.send(("%s\r\n" % domain).encode('utf-8'))
    buff = b""
    while True:
        data = sock.recv(1024)
        if len(data) == 0:
            break
        buff += data
    return buff.decode('utf-8')


def check_domain_candidate(keyword, args):
    """
    'Builds' the domain name (just appends .com as that's all I check right now)
    then checks its WHOIS status - assuming we don't skip this in the args.
    """
    domain_name = keyword + '.com'

    if not args.skip_whois:
        whois_response = whois_request(domain_name)

        if re.search("Domain Name: %s" % domain_name.upper(), whois_response):
            # we have a match; the domain is already registered
            if args.show_taken:
                try:
                    expiry_date = re.search("Registry Expiry Date\: (.*)", whois_response).group(1)
                    print(domain_name + ' is NOT available; expiry date is ' + expiry_date)
                except AttributeError:
                    print(domain_name + ' is NOT available')
        else:
            print(domain_name + ' is available')
    else:
        print(domain_name + ' was generated (but not checked for availability)')


def parse_cli_args():
    """ Parses the command line arguments to (e.g.) keep duplicate combinations. """
    parser = argparse.ArgumentParser()

    parser.add_argument(
        '--skip-whois', default=False,
        help='Only generate the possible domain name candidates (and output them)'
             '- this does not do the actual WHOIS availability check.',
        action='store_true'
    )

    parser.add_argument(
        '--show-taken', default=False,
        help='Outputs all results, even generated domains which have already been registered.',
        action='store_true'
    )

    parser.add_argument(
        '--starts-with',
        help='Add the submitted string to the start of every generated keyword.',
        required=False)

    parser.add_argument(
        '--ends-with',
        help='Add the submitted string to the end of every generated keyword.',
        required=False)

    parser.add_argument(
        '--kws', nargs='+',
        help='<Required> Provide one or more groups of keywords (each group can be a single keyword'
             'or comma separated, e.g. "--kws kw1,kw1alt,kw1b kw2,kw2alt"'
             'and "--kws kw1 kw2" are both valid inputs)',
        required=True)

    return parser.parse_args()


if __name__ == "__main__":
    args = parse_cli_args()

    kws = [kw.split(",") for kw in args.kws]

    # build all combinations from the provided keyword groups (i.e. the list of list of strings)
    combinations = list(itertools.product(*kws))

    for kw_tuple in combinations:
        kw = ''.join(str(i) for i in kw_tuple)
        kw = args.starts_with + kw if args.starts_with else kw
        kw = kw + args.ends_with if args.ends_with else kw
        check_domain_candidate(kw, args)

```

## Git Repository

```plaintext
origin	https://github.com/TristanPerry/domain-name-generator.git (fetch)
origin	https://github.com/TristanPerry/domain-name-generator.git (push)
```
