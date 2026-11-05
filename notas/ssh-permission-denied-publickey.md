# SSH: "Permission denied (publickey)" con la clave correcta

**Síntoma:** la clave SSH es válida y funciona contra otros servidores, pero este responde `Permission denied (publickey)` al instante, sin pedir contraseña (con `PasswordAuthentication no`).

## Diagnóstico

1. Cliente verboso: qué claves ofrece y qué responde el servidor:

```bash
ssh -v usuario@host                   # -vvv si hace falta el detalle fino
ssh-add -l                            # qué claves tiene el agente
```

2. En el servidor, sshd cuenta por qué rechaza (la razón más común está ahí, no en el cliente):

```bash
journalctl -u sshd -n 20              # "Authentication refused: bad ownership or modes"
sudo sshd -T | grep -iE 'pubkey|authorizedkeys|password'
ls -la /home/usuario/.ssh             # permisos y dueño
```

## Causas habituales

- Permisos abiertos en el servidor: `~/.ssh` debe ser `700` y `authorized_keys` `600`. Con permisos de grupo/otros, sshd rechaza la clave en silencio.
- `authorized_keys` con dueño equivocado (copiado o editado como root).
- El home con permiso de escritura para grupo.
- Usuario o ruta equivocados (`root@` por defecto, `AuthorizedKeysFile` custom).
- El agente no ofrece la clave que el servidor espera.

## Solución

En el servidor:

```bash
chown -R usuario:usuario /home/usuario/.ssh
chmod 700 /home/usuario/.ssh
chmod 600 /home/usuario/.ssh/authorized_keys
chmod g-w /home/usuario               # home sin escritura de grupo
```

En el cliente, el método que lo deja todo bien de serie:

```bash
ssh-copy-id usuario@host
```

**Ojo:** en distros con SELinux (RHEL/Fedora) hace falta además `restorecon -Rv ~/.ssh`; en Arch no aplica.

## Prevención

- Usar `ssh-copy-id` en vez de copiar la clave a mano.
- No editar nunca `authorized_keys` con sudo.
