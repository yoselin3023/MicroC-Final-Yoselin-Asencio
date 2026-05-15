# MicroC-Final-Yoselin-Asencio
MicroC Automatas Practica Final
"""
============================================================
  MicroC — Compilador / Analizador Léxico
  Curso  : AUTOMATAS Y LENGUAJES  2026
  Semestre: V   Carrera: Ing. Sistemas
  Catedrático: Ing. Baudilio Boteo

  ✔ FASE I   — Tokens básicos, símbolos, identificadores
  ✔ FASE II  — Directivas, librerías, números, comentarios
  ✔ EXTRAS   — Proceso detallado de reconocimiento visible,
                resaltado de sintaxis, tabla de símbolos,
                columnas
  ✔ FIX      — -0 reconocido como ERROR léxico
                (negativo de cero no tiene sentido semántico)
============================================================
"""

import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import os

# ╔══════════════════════════════════════════════════════════╗
# ║  CLASE: UnidadesLexicas                                  ║
# ╚══════════════════════════════════════════════════════════╝
class UnidadesLexicas:

    def __init__(self):
        self.Palabra = {}
        self.Simbolo = {}
        self._cargar_simbolos()
        self._cargar_palabras()

    def _cargar_simbolos(self):
        self.Simbolo['(']  = 75;  self.Simbolo[')']  = 76
        self.Simbolo['{']  = 77;  self.Simbolo['}']  = 78
        self.Simbolo['[']  = 79;  self.Simbolo[']']  = 80
        self.Simbolo['+']  = 81;  self.Simbolo['-']  = 82
        self.Simbolo['*']  = 83;  self.Simbolo['/']  = 84
        self.Simbolo['%']  = 85
        self.Simbolo['<']  = 86;  self.Simbolo['>']  = 87
        self.Simbolo['=='] = 88;  self.Simbolo['!='] = 89
        self.Simbolo['<='] = 90;  self.Simbolo['>='] = 91
        self.Simbolo[';']  = 92;  self.Simbolo[',']  = 93
        self.Simbolo['.']  = 94;  self.Simbolo['#']  = 95
        self.Simbolo['"']  = 96;  self.Simbolo["'"]  = 97
        self.Simbolo['\\'] = 98;  self.Simbolo['@']  = 99
        self.Simbolo['=']  = 100; self.Simbolo['++'] = 101
        self.Simbolo['--'] = 102; self.Simbolo['+='] = 103
        self.Simbolo['-='] = 104; self.Simbolo['*='] = 105
        self.Simbolo['/='] = 106
        self.Simbolo['&&'] = 107; self.Simbolo['||'] = 108
        self.Simbolo['!']  = 109
        self.Simbolo['&']  = 115; self.Simbolo['|']  = 116
        self.Simbolo['^']  = 117; self.Simbolo['~']  = 118
        self.Simbolo['<<'] = 119; self.Simbolo['>>'] = 120
        self.Simbolo[':']  = 113; self.Simbolo['?']  = 114

    def _cargar_palabras(self):
        for p,t in [('auto',1),('break',2),('case',3),('char',4),
            ('const',5),('continue',6),('default',7),('do',8),
            ('double',9),('else',10),('enum',11),('extern',12),
            ('float',13),('for',14),('goto',15),('if',16),
            ('int',17),('long',18),('register',19),('return',20),
            ('short',21),('signed',22),('sizeof',23),('static',24),
            ('struct',25),('switch',26),('typedef',27),('union',28),
            ('unsigned',29),('void',30),('volatile',31),('while',32),
        ]: self.Palabra[p] = t
        for p,t in [('class',33),('public',34),('private',35),('protected',36),
            ('new',37),('delete',38),('this',39),('true',40),
            ('false',41),('namespace',42),('using',43),('template',44),
            ('virtual',45),('bool',46),('string',47),('nullptr',48),
        ]: self.Palabra[p] = t
        for p,t in [('include',50),('define',51),('ifdef',52),('ifndef',53),
            ('endif',54),('undef',55),('pragma',56),('error',57),
            ('warning',58),('elif',59),
        ]: self.Palabra[p] = t
        for p,t in [('printf',60),('scanf',61),('fprintf',62),('fscanf',63),
            ('fopen',64),('fclose',65),('fgets',66),('fputs',67),
            ('feof',68),('fflush',69),
        ]: self.Palabra[p] = t
        for p,t in [('malloc',70),('calloc',71),('realloc',72),
            ('free',73),('exit',74),
        ]: self.Palabra[p] = t
        for p,t in [('strcpy',121),('strncpy',122),('strcat',123),('strncat',124),
            ('strcmp',125),('strncmp',126),('strlen',127),('strchr',128),
            ('strstr',129),('strtok',130),
        ]: self.Palabra[p] = t
        for p,t in [('getch',131),('getche',132),('putch',133),('clrscr',134),
            ('gotoxy',135),('kbhit',136),('textcolor',137),
            ('textbackground',138),('cprintf',139),
        ]: self.Palabra[p] = t
        for p,t in [('sqrt',140),('pow',141),('fabs',142),('ceil',143),
            ('floor',144),('sin',145),('cos',146),('tan',147),
            ('log',148),('exp',149),
        ]: self.Palabra[p] = t
        for p,t in [('cout',150),('cin',151),('endl',152),('std',153),
            ('vector',154),('map',155),('set',156),('pair',157),
            ('list',158),('queue',159),
        ]: self.Palabra[p] = t

    def GetTokenSimbolo(self, Lexema):
        return self.Simbolo.get(Lexema, -1)

    def GetTokenPalabra(self, Lexema):
        return self.Palabra.get(Lexema, 300)


# ╔══════════════════════════════════════════════════════════╗
# ║  CLASE: AnalizadorLexico                                 ║
# ║  Guarda proceso detallado de cada reconocimiento         ║
# ╚══════════════════════════════════════════════════════════╝
class AnalizadorLexico:

    def __init__(self):
        self.Lista      : list = []
        self.cont       : int  = 0
        self.Linea      : int  = 1
        self._col       : int  = 1

    def GetAlfabetoAlfanumerico(self, c): return 1 if (c.isalpha() or c=='_') else 0
    def GetAlfabetoNumero(self, c):       return 1 if c.isdigit() else 0
    def GetAlfabetoSimbolo(self, c):
        return 1 if c in set('(){}[]+-*/%<>=!&|^~;,."\'\\@:?#') else 0

    def _add(self, lin, col, lex, tok, tipo, proceso):
        self.Lista.append((lin, col, lex, tok, tipo, proceso))

    # ── NUEVO: Detecta si un '-' seguido de '0' (sin más dígitos ni punto) ──
    # ── es el caso "-0", que se registra como ERROR léxico.                 ──
    def _es_negativo_cero(self, Archivo, pos_despues_del_cero):
        """
        Retorna True si la secuencia que sigue al '-' ya consumido
        es exactamente '0' (posiblemente con sufijos l/u/f) y
        el resultado semántico sería -0.
        El '-' ya fue consumido; pos_despues_del_cero apunta al '0'.
        """
        i = pos_despues_del_cero
        if i >= len(Archivo) or Archivo[i] != '0':
            return False
        i += 1
        # Saltear sufijos válidos (l, u, f, L, U, F)
        while i < len(Archivo) and Archivo[i].lower() in 'luf':
            i += 1
        # El siguiente carácter NO debe ser dígito, punto ni letra
        # (si lo fuera, sería -01 = -octal o -0.5 = real, que son válidos)
        if i < len(Archivo):
            c = Archivo[i]
            if c.isdigit() or c == '.' or c.isalpha() or c == '_':
                return False
        return True

    def _contexto_permite_unario(self):
        """
        Retorna True cuando el '-' puede interpretarse como operador
        unario (signo), es decir: al inicio del código, o después de
        un operador / símbolo de apertura / asignación.
        Tokens que admiten operador unario después de ellos:
          - ningún token aún (inicio de archivo)
          - símbolo: ( [ , ; = += -= *= /= == != < > <= >= && || ! & | ^ ~ << >>
        """
        if not self.Lista:
            return True
        ultimo_tipo  = self.Lista[-1][4]
        ultimo_tok   = self.Lista[-1][3]
        ultimo_lex   = self.Lista[-1][2]
        # Después de cualquier operador o símbolo de apertura → unario posible
        if ultimo_tipo == 'SIMBOLO':
            # NO es unario después de ) ] ++ --  (esos son postfix sobre expresión)
            if ultimo_lex in (')', ']', '++', '--'):
                return False
            return True
        # Después de palabras reservadas como return, if, while, etc. → unario posible
        if ultimo_tipo in ('RESERVADA', 'RESERVADA_CPP'):
            return True
        return False

    def IdentificadorPalabraReservada(self, Archivo, UL):
        lin = self.Linea; col = self._col
        Lexema = ''; pasos = []

        pasos.append(f'1. Carácter inicial "{Archivo[self.cont]}" detectado como LETRA/GUIÓN_BAJO')
        pasos.append(f'   → Entra al autómata: IdentificadorPalabraReservada')

        while self.cont < len(Archivo):
            c = Archivo[self.cont]
            if c.isalnum() or c == '_':
                Lexema += c; self.cont += 1; self._col += 1
            else:
                break

        pasos.append(f'2. Lexema construido carácter a carácter: "{Lexema}"')
        pasos.append(f'   → Consumió {len(Lexema)} carácter(es), detuvo en: "{Archivo[self.cont] if self.cont < len(Archivo) else "FIN"}"')

        token = UL.GetTokenPalabra(Lexema)
        pasos.append(f'3. Búsqueda en tabla de palabras: GetTokenPalabra("{Lexema}")')

        if token == 300:
            tipo = 'IDENTIFICADOR'
            pasos.append(f'   → "{Lexema}" NO está en la tabla de palabras reservadas')
            pasos.append(f'   → Retorna token = 300  →  Clasificado como: IDENTIFICADOR')
        else:
            if   1  <= token <= 32:   tipo = 'RESERVADA'
            elif 33 <= token <= 48:   tipo = 'RESERVADA_CPP'
            elif 50 <= token <= 59:   tipo = 'DIRECTIVA'
            elif 60 <= token <= 69:   tipo = 'FUNC_STDIO'
            elif 70 <= token <= 74:   tipo = 'FUNC_STDLIB'
            elif 121 <= token <= 130: tipo = 'FUNC_STRING'
            elif 131 <= token <= 139: tipo = 'FUNC_CONIO'
            elif 140 <= token <= 149: tipo = 'FUNC_MATH'
            elif 150 <= token <= 159: tipo = 'RESERVADA_CPP'
            else:                     tipo = 'RESERVADA'
            pasos.append(f'   → "{Lexema}" ENCONTRADO en la tabla con token = {token}')
            pasos.append(f'   → Clasificado como: {tipo}')

        pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:{token}  Tipo:{tipo}')
        self._add(lin, col, Lexema, token, tipo, pasos)

    def EnteroReal(self, Archivo):
        lin = self.Linea; col = self._col
        pasos = []
        pasos.append(f'1. Carácter inicial "{Archivo[self.cont]}" detectado como DÍGITO')
        pasos.append(f'   → Entra al autómata: EnteroReal')

        if (Archivo[self.cont]=='0' and self.cont+1<len(Archivo)
                and Archivo[self.cont+1].lower()=='x'):
            pasos.append(f'2. Detectado prefijo "0x" → Número HEXADECIMAL')
            Lexema='0x'; self.cont+=2; self._col+=2
            while self.cont<len(Archivo) and Archivo[self.cont] in '0123456789abcdefABCDEF':
                Lexema+=Archivo[self.cont]; self.cont+=1; self._col+=1
            pasos.append(f'3. Dígitos hexadecimales consumidos: "{Lexema}"')
            pasos.append(f'   → Token = 202  →  Tipo: HEXADECIMAL')
            pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:202  Tipo:HEXADECIMAL')
            self._add(lin,col,Lexema,202,'HEXADECIMAL',pasos); return

        if (Archivo[self.cont]=='0' and self.cont+1<len(Archivo)
                and Archivo[self.cont+1].lower()=='b'):
            pasos.append(f'2. Detectado prefijo "0b" → Número BINARIO')
            Lexema='0b'; self.cont+=2; self._col+=2
            while self.cont<len(Archivo) and Archivo[self.cont] in '01':
                Lexema+=Archivo[self.cont]; self.cont+=1; self._col+=1
            pasos.append(f'3. Dígitos binarios (0,1) consumidos: "{Lexema}"')
            pasos.append(f'   → Token = 203  →  Tipo: BINARIO')
            pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:203  Tipo:BINARIO')
            self._add(lin,col,Lexema,203,'BINARIO',pasos); return

        if (Archivo[self.cont]=='0' and self.cont+1<len(Archivo)
                and Archivo[self.cont+1] in '01234567'):
            pasos.append(f'2. Detectado "0" seguido de dígito 0-7 → Número OCTAL')
            Lexema='0'; self.cont+=1; self._col+=1
            while self.cont<len(Archivo) and Archivo[self.cont] in '01234567':
                Lexema+=Archivo[self.cont]; self.cont+=1; self._col+=1
            pasos.append(f'3. Dígitos octales (0-7) consumidos: "{Lexema}"')
            pasos.append(f'   → Token = 204  →  Tipo: OCTAL')
            pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:204  Tipo:OCTAL')
            self._add(lin,col,Lexema,204,'OCTAL',pasos); return

        pasos.append(f'2. No es hex/bin/oct → Autómata de entero/real')
        Lexema=''; es_punto=False
        while self.cont<len(Archivo):
            c=Archivo[self.cont]
            if c.isdigit():
                Lexema+=c; self.cont+=1; self._col+=1
            elif c=='.' and not es_punto:
                sig=Archivo[self.cont+1] if self.cont+1<len(Archivo) else ''
                if sig.isdigit():
                    es_punto=True; Lexema+=c; self.cont+=1; self._col+=1
                    pasos.append(f'   → Encontrado punto decimal "." con dígito siguiente → cambia a REAL')
                else: break
            else: break

        sufijo=''
        while self.cont<len(Archivo) and Archivo[self.cont].lower() in 'lfu':
            sufijo+=Archivo[self.cont]; Lexema+=Archivo[self.cont]
            self.cont+=1; self._col+=1
        if sufijo:
            pasos.append(f'   → Sufijo detectado: "{sufijo}"')

        if not Lexema: return

        token = 201 if es_punto else 200
        tipo  = 'REAL' if es_punto else 'ENTERO'
        pasos.append(f'3. Lexema numérico construido: "{Lexema}"')
        if es_punto:
            pasos.append(f'   → Contiene punto decimal → Token = 201 → Tipo: REAL')
        else:
            pasos.append(f'   → Sin punto decimal → Token = 200 → Tipo: ENTERO')
        pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:{token}  Tipo:{tipo}')
        self._add(lin,col,Lexema,token,tipo,pasos)

    def AutomataComentario(self, Archivo):
        lin=self.Linea; col=self._col-1; pasos=[]
        if self.cont>=len(Archivo):
            pasos.append('1. "/" al final del archivo → operador división')
            pasos.append(f'   → Token = 84  →  Tipo: SIMBOLO')
            self._add(lin,col,'/',84,'SIMBOLO',pasos); return
        sig=Archivo[self.cont]
        pasos.append(f'1. Carácter "/" detectado')
        pasos.append(f'   → Siguiente carácter: "{sig}"')
        if sig=='/':
            pasos.append(f'2. "//" → Autómata de COMENTARIO DE LÍNEA')
            self.cont+=1; self._col+=1; contenido=''
            while self.cont<len(Archivo):
                c=Archivo[self.cont]; self.cont+=1
                if c=='\n': self.Linea+=1; self._col=1; break
                else: self._col+=1; contenido+=c
            pasos.append(f'3. Contenido del comentario: "{contenido.strip()}"')
            pasos.append(f'   → Token = 400  →  Tipo: COMENTARIO_LINEA')
            pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Token:400  Tipo:COMENTARIO_LINEA')
            self._add(lin,col,f'//{contenido.strip()}',400,'COMENTARIO_LINEA',pasos)
        elif sig=='*':
            pasos.append(f'2. "/*" → Autómata de COMENTARIO DE BLOQUE')
            self.cont+=1; self._col+=1; contenido=''; lini=self.Linea
            while self.cont<len(Archivo)-1:
                c=Archivo[self.cont]
                if c=='\n': self.Linea+=1; self._col=1; contenido+=' '
                elif c=='*' and Archivo[self.cont+1]=='/':
                    self.cont+=2; self._col+=2; break
                else: self._col+=1; contenido+=c
                self.cont+=1
            pasos.append(f'3. Contenido: "{contenido.strip()}"')
            pasos.append(f'   → Token = 401  →  Tipo: COMENTARIO_BLOQUE')
            pasos.append(f'4. RESULTADO → Línea:{lini}  Col:{col}  Token:401  Tipo:COMENTARIO_BLOQUE')
            self._add(lini,col,f'/*{contenido.strip()}*/',401,'COMENTARIO_BLOQUE',pasos)
        else:
            pasos.append(f'2. Siguiente carácter "{sig}" no es "/" ni "*"')
            pasos.append(f'   → "/" es operador de DIVISIÓN → Token = 84')
            pasos.append(f'3. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"/"  Token:84  Tipo:SIMBOLO')
            self._add(lin,col,'/',84,'SIMBOLO',pasos)

    def AutomataCadena(self, Archivo):
        lin=self.Linea; col=self._col; pasos=[]
        pasos.append(f'1. Comilla doble " detectada en Línea:{lin} Col:{col}')
        pasos.append(f'   → Entra al autómata: Cadena de texto')
        self.cont+=1; self._col+=1; contenido='"'
        while self.cont<len(Archivo):
            c=Archivo[self.cont]; self.cont+=1
            if c=='\\' and self.cont<len(Archivo):
                esc=Archivo[self.cont]; contenido+='\\'+esc
                self.cont+=1; self._col+=2
                pasos.append(f'   → Carácter de escape detectado: "\\{esc}"')
            elif c=='"':
                contenido+='"'; self._col+=1; break
            elif c=='\n':
                self.Linea+=1; self._col=1; contenido+=c
            else:
                contenido+=c; self._col+=1
        pasos.append(f'2. Cadena completa: {contenido}')
        pasos.append(f'   → Token = 500  →  Tipo: CADENA')
        pasos.append(f'3. RESULTADO → Línea:{lin}  Col:{col}  Token:500  Tipo:CADENA')
        self._add(lin,col,contenido,500,'CADENA',pasos)

    def AutomataCaracter(self, Archivo):
        lin=self.Linea; col=self._col; pasos=[]
        pasos.append(f'1. Comilla simple \' detectada en Línea:{lin} Col:{col}')
        pasos.append(f'   → Entra al autómata: Carácter literal')
        self.cont+=1; self._col+=1; contenido="'"
        while self.cont<len(Archivo):
            c=Archivo[self.cont]; self.cont+=1
            if c=='\\' and self.cont<len(Archivo):
                esc=Archivo[self.cont]; contenido+='\\'+esc
                self.cont+=1; self._col+=2
            elif c=="'":
                contenido+="'"; self._col+=1; break
            else:
                contenido+=c; self._col+=1
        pasos.append(f'2. Carácter literal: {contenido}')
        pasos.append(f'   → Token = 501  →  Tipo: CARACTER')
        pasos.append(f'3. RESULTADO → Línea:{lin}  Col:{col}  Token:501  Tipo:CARACTER')
        self._add(lin,col,contenido,501,'CARACTER',pasos)

    def AutomataDirectiva(self, Archivo):
        UL=UnidadesLexicas()
        lin=self.Linea; col=self._col-1; nombre=''; pasos=[]
        pasos.append(f'1. Carácter "#" detectado → Autómata de DIRECTIVA')
        while self.cont<len(Archivo) and (Archivo[self.cont].isalpha() or Archivo[self.cont]=='_'):
            nombre+=Archivo[self.cont]; self.cont+=1; self._col+=1
        token=UL.GetTokenPalabra(nombre)
        pasos.append(f'2. Nombre de directiva leído: "{nombre}"')
        pasos.append(f'   → GetTokenPalabra("{nombre}") = {token}')
        pasos.append(f'   → Token = {token}  →  Tipo: DIRECTIVA')
        pasos.append(f'3. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"#{nombre}"  Token:{token}  Tipo:DIRECTIVA')
        self._add(lin,col,f'#{nombre}',token,'DIRECTIVA',pasos)

        while self.cont<len(Archivo) and Archivo[self.cont]==' ':
            self.cont+=1; self._col+=1
        if self.cont<len(Archivo):
            c=Archivo[self.cont]
            if c=='<':
                col2=self._col; arg='<'; self.cont+=1; self._col+=1
                while self.cont<len(Archivo) and Archivo[self.cont]!='>':
                    arg+=Archivo[self.cont]; self.cont+=1; self._col+=1
                if self.cont<len(Archivo):
                    arg+='>'; self.cont+=1; self._col+=1
                p2=['1. Argumento de directiva con "<...>"',
                    f'   → Argumento leído: "{arg}"',
                    f'   → Token = 502  →  Tipo: ARG_DIRECTIVA']
                self._add(lin,col2,arg,502,'ARG_DIRECTIVA',p2)
            elif c=='"':
                self.AutomataCadena(Archivo)
                if self.Lista:
                    l,c2,lx,tk,tp,pr=self.Lista[-1]
                    self.Lista[-1]=(l,c2,lx,502,'ARG_DIRECTIVA',pr)

    # ══════════════════════════════════════════════════════════
    # NUEVO AUTÓMATA: NegativoCero
    # Detecta la secuencia  -0  (con posibles sufijos l/u/f)
    # cuando el '-' actúa como operador unario y el operando
    # es exactamente cero.  Emite un token ERROR.
    # ══════════════════════════════════════════════════════════
    def AutomataNegativoCero(self, Archivo, col_menos):
        """
        Se llama cuando ya se detectó que el '-' en col_menos
        es un operador unario seguido de '0' puro.
        Consume el '0' (y sufijos opcionales) y emite ERROR.
        """
        lin = self.Linea
        pasos = []
        pasos.append(f'1. Carácter "-" detectado en Línea:{lin} Col:{col_menos}')
        pasos.append(f'   → Siguiente carácter: "0"')
        pasos.append(f'2. Autómata: NegativoCero')
        pasos.append(f'   → El contexto indica operador UNARIO ("-" sin expresión izquierda válida)')
        pasos.append(f'   → Se lee el operando: "0"')

        # Consumir el '0'
        Lexema = '-0'
        self.cont += 1   # consume el '0'
        self._col += 1

        # Consumir sufijos opcionales
        sufijos = ''
        while self.cont < len(Archivo) and Archivo[self.cont].lower() in 'luf':
            sufijos += Archivo[self.cont]
            Lexema  += Archivo[self.cont]
            self.cont += 1; self._col += 1
        if sufijos:
            pasos.append(f'   → Sufijo detectado: "{sufijos}"  (incluido en el lexema del error)')

        pasos.append(f'3. ⚠ ANÁLISIS SEMÁNTICO LÉXICO:')
        pasos.append(f'   → "-0" es el negativo de cero.')
        pasos.append(f'   → En aritmética, -0 == 0 (enteros) o -0.0 (IEEE 754 flotante).')
        pasos.append(f'   → Su uso como literal es innecesario y potencialmente')
        pasos.append(f'     confuso; ningún compilador lo genera intencionalmente.')
        pasos.append(f'   → Token = ERR  →  Tipo: ERROR_NEGATIVO_CERO')
        pasos.append(f'4. RESULTADO → ⚠ Línea:{lin}  Col:{col_menos}  '
                     f'Lexema:"{Lexema}"  Token:ERR  Tipo:ERROR_NEGATIVO_CERO')
        self._add(lin, col_menos, Lexema, 'ERR', 'ERROR_NEGATIVO_CERO', pasos)

    def _procesar_simbolo(self, Archivo, UL):
        lin=self.Linea; col=self._col; pasos=[]
        c_actual = Archivo[self.cont]
        pasos.append(f'1. Carácter "{c_actual}" detectado como SÍMBOLO')

        # ── CHECK ESPECIAL: ¿es un '-' que formaría '-0'? ──────────────────
        if c_actual == '-':
            # Verificar que no sea parte de '-=' ni '--'
            doble_candidato = ''
            if self.cont + 1 < len(Archivo):
                doble_candidato = c_actual + Archivo[self.cont + 1]

            es_operador_compuesto = UL.GetTokenSimbolo(doble_candidato) != -1 and doble_candidato in ('-=', '--')

            if not es_operador_compuesto and self._contexto_permite_unario():
                pos_sig = self.cont + 1
                if self._es_negativo_cero(Archivo, pos_sig):
                    col_menos = self._col
                    self.cont += 1   # consume el '-'
                    self._col += 1
                    self.AutomataNegativoCero(Archivo, col_menos)
                    return
        # ───────────────────────────────────────────────────────────────────

        if self.cont+1<len(Archivo):
            doble=Archivo[self.cont]+Archivo[self.cont+1]
            tok=UL.GetTokenSimbolo(doble)
            if tok!=-1:
                pasos.append(f'2. Intento símbolo doble: "{doble}" → GetTokenSimbolo = {tok}  ✔ encontrado')
                pasos.append(f'   → Token = {tok}  →  Tipo: SIMBOLO')
                pasos.append(f'3. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{doble}"  Token:{tok}  Tipo:SIMBOLO')
                self._add(lin,col,doble,tok,'SIMBOLO',pasos)
                self.cont+=2; self._col+=2; return
            else:
                pasos.append(f'2. Intento símbolo doble: "{doble}" → no encontrado, se prueba simple')
        Lexema=Archivo[self.cont]
        tok=UL.GetTokenSimbolo(Lexema)
        if tok!=-1:
            pasos.append(f'3. Símbolo simple: "{Lexema}" → GetTokenSimbolo = {tok}  ✔ encontrado')
            pasos.append(f'   → Token = {tok}  →  Tipo: SIMBOLO')
            pasos.append(f'4. RESULTADO → Línea:{lin}  Col:{col}  Lexema:"{Lexema}"  Token:{tok}  Tipo:SIMBOLO')
            self._add(lin,col,Lexema,tok,'SIMBOLO',pasos)
        else:
            pasos.append(f'3. "{Lexema}" NO encontrado en tabla de símbolos')
            pasos.append(f'   → Token = ERR  →  Tipo: ERROR')
            pasos.append(f'4. RESULTADO → ⚠ SIMBOLO NO ENCONTRADO')
            self._add(lin,col,Lexema,'ERR','ERROR',pasos)
        self.cont+=1; self._col+=1

    def AnalisisLexico(self, Archivo: str) -> list:
        self.Lista=[]; self.cont=0; self.Linea=1; self._col=1
        UL=UnidadesLexicas()
        while self.cont<len(Archivo):
            c=Archivo[self.cont]
            if   self.GetAlfabetoAlfanumerico(c): self.IdentificadorPalabraReservada(Archivo,UL)
            elif self.GetAlfabetoNumero(c):        self.EnteroReal(Archivo)
            elif c=='/':  self.cont+=1; self._col+=1; self.AutomataComentario(Archivo)
            elif c=='#':  self.cont+=1; self._col+=1; self.AutomataDirectiva(Archivo)
            elif c=='"':  self.AutomataCadena(Archivo)
            elif c=="'":  self.AutomataCaracter(Archivo)
            elif c in (' ','\t','\r','\x00'): self.cont+=1; self._col+=1
            elif c=='\n': self.Linea+=1; self.cont+=1; self._col=1
            elif self.GetAlfabetoSimbolo(c): self._procesar_simbolo(Archivo,UL)
            else:
                p=[f'1. Carácter "{c}" no pertenece a ningún alfabeto reconocido',
                   f'   → SIMBOLO NO ENCONTRADO  →  Token: ERR']
                self._add(self.Linea,self._col,c,'ERR','ERROR',p)
                self.cont+=1; self._col+=1
        return self.Lista


# ╔══════════════════════════════════════════════════════════╗
# ║  CLASE: frmEditor                                        ║
# ╚══════════════════════════════════════════════════════════╝
class frmEditor(tk.Tk):

    C = {
        'bg'           : '#0a1628',
        'surface'      : '#0f2040',
        'surface2'     : '#132850',
        'border'       : '#1e3a6e',
        'accent'       : '#1e90ff',
        'accent_h'     : '#4dabff',
        'btn_bg'       : '#163060',
        'btn_fg'       : '#90bfff',
        'txt'          : '#cce0ff',
        'muted'        : '#4a6fa0',
        'editor_bg'    : '#071020',
        'sel'          : '#1a3d7a',
        'hdr'          : '#f0c040',
        'tok_reserv'   : '#ff9d5c',
        'tok_id'       : '#cce0ff',
        'tok_sym'      : '#52d9ff',
        'tok_num'      : '#a8ff78',
        'tok_err'      : '#ff5f72',
        'tok_linea'    : '#4a6fa0',
        'tok_lex'      : '#90bfff',
        'tok_directiva': '#c792ea',
        'tok_func'     : '#82aaff',
        'tok_comment'  : '#546e7a',
        'tok_string'   : '#c3e88d',
        'tok_char'     : '#f78c6c',
        'tok_hex'      : '#ffcb6b',
        'tok_cpp'      : '#89ddff',
        'proc_step'    : '#4a6fa0',
        'proc_arrow'   : '#1e90ff',
        'proc_ok'      : '#a8ff78',
        'proc_err'     : '#ff5f72',
        'proc_result'  : '#f0c040',
    }

    SYNTAX_TAGS = {
        'RESERVADA':'#ff9d5c','RESERVADA_CPP':'#89ddff',
        'DIRECTIVA':'#c792ea','FUNC_STDIO':'#82aaff',
        'FUNC_STDLIB':'#82aaff','FUNC_STRING':'#82aaff',
        'FUNC_CONIO':'#82aaff','FUNC_MATH':'#82aaff',
        'ENTERO':'#a8ff78','REAL':'#a8ff78',
        'HEXADECIMAL':'#ffcb6b','OCTAL':'#ffcb6b','BINARIO':'#ffcb6b',
        'CADENA':'#c3e88d','CARACTER':'#f78c6c',
        'COMENTARIO_LINEA':'#546e7a','COMENTARIO_BLOQUE':'#546e7a',
        'SIMBOLO':'#52d9ff','ERROR':'#ff5f72',
        'ERROR_NEGATIVO_CERO':'#ff5f72',   # ← nuevo tipo visible en resaltado
    }

    def __init__(self):
        super().__init__()
        self.Archivo   = ''
        self._tokens   = []
        self._hl_after = None
        self._cfg_ventana()
        self._build_menu()
        self._build_ui()

    def _cfg_ventana(self):
        self.title('MicroC  —  Analizador Léxico')
        self.geometry('1400x800')
        self.minsize(1000, 600)
        self.configure(bg=self.C['bg'])
        try: self.state('zoomed')
        except: pass

    def _build_menu(self):
        C=self.C
        mb=tk.Menu(self,bg=C['surface'],fg=C['txt'],
                   activebackground=C['accent'],activeforeground='white',tearoff=False)
        ma=tk.Menu(mb,tearoff=False,bg=C['surface'],fg=C['txt'],
                   activebackground=C['accent'],activeforeground='white')
        ma.add_command(label='Nuevo         Ctrl+N', command=self.OpcNuevo_Click)
        ma.add_command(label='Abrir...      Ctrl+A', command=self.OpcAbrir_Click)
        ma.add_separator()
        ma.add_command(label='Guardar       Ctrl+G', command=self.OpcGuardar_Click)
        ma.add_command(label='Guardar Como...', command=self.OpcGuardarComo_Click)
        ma.add_separator()
        ma.add_command(label='Salir', command=self.OpcSalir_Click)
        mb.add_cascade(label='Archivos', menu=ma)
        mc=tk.Menu(mb,tearoff=False,bg=C['surface'],fg=C['txt'],
                   activebackground=C['accent'],activeforeground='white')
        mc.add_command(label='Compilar      F5', command=self.compilarToolStripMenuItem_Click)
        mc.add_command(label='Limpiar       F6', command=self._limpiar_tokens)
        mb.add_cascade(label='Compilar', menu=mc)
        mh=tk.Menu(mb,tearoff=False,bg=C['surface'],fg=C['txt'],
                   activebackground=C['accent'],activeforeground='white')
        mh.add_command(label='Acerca de MicroC', command=self._acerca)
        mb.add_cascade(label='Ayuda', menu=mh)
        self.config(menu=mb)
        self.bind('<Control-n>', lambda e: self.OpcNuevo_Click())
        self.bind('<Control-N>', lambda e: self.OpcNuevo_Click())
        self.bind('<Control-a>', lambda e: self.OpcAbrir_Click())
        self.bind('<Control-A>', lambda e: self.OpcAbrir_Click())
        self.bind('<Control-g>', lambda e: self.OpcGuardar_Click())
        self.bind('<Control-G>', lambda e: self.OpcGuardar_Click())
        self.bind('<F5>',        lambda e: self.compilarToolStripMenuItem_Click())
        self.bind('<F6>',        lambda e: self._limpiar_tokens())

    def _build_ui(self):
        C=self.C

        # TOOLBAR
        tb=tk.Frame(self,bg=C['surface'],pady=6,padx=8)
        tb.pack(fill='x',side='top')
        def btn(p,t,cmd,primary=False):
            bg=C['accent'] if primary else C['btn_bg']
            fg='white'     if primary else C['btn_fg']
            bh=C['accent_h'] if primary else C['border']
            b=tk.Label(p,text=t,bg=bg,fg=fg,
                       font=('Consolas',9,'bold' if primary else 'normal'),
                       padx=10,pady=4,cursor='hand2',relief='flat')
            b.pack(side='left',padx=2)
            b.bind('<Button-1>',lambda e: cmd())
            b.bind('<Enter>',   lambda e: b.config(bg=bh))
            b.bind('<Leave>',   lambda e: b.config(bg=bg))
            return b
        btn(tb,'📄 Nuevo',        self.OpcNuevo_Click)
        btn(tb,'📂 Abrir',        self.OpcAbrir_Click)
        btn(tb,'💾 Guardar',      self.OpcGuardar_Click)
        btn(tb,'💾 Guardar Como', self.OpcGuardarComo_Click)
        tk.Frame(tb,bg=C['border'],width=1).pack(side='left',fill='y',padx=8,pady=2)
        btn(tb,'▶ Compilar F5',   self.compilarToolStripMenuItem_Click,primary=True)
        tk.Label(tb,text='MicroC 2026  —  FASE I + II + EXTRAS',
                 bg=C['surface'],fg=C['muted'],font=('Consolas',8)).pack(side='right',padx=10)
        tk.Frame(self,bg=C['border'],height=1).pack(fill='x')

        # PANED principal
        paned=tk.PanedWindow(self,orient='horizontal',bg=C['border'],
                             sashwidth=5,sashrelief='flat')
        paned.pack(fill='both',expand=True)

        # ── PANEL IZQUIERDO: editor ───────────────────────
        lf=tk.Frame(paned,bg=C['bg'])
        paned.add(lf,minsize=350,stretch='always')
        tk.Frame(lf,bg=C['border'],height=1).pack(fill='x')
        hl=tk.Frame(lf,bg=C['surface2'],pady=4)
        hl.pack(fill='x')
        tk.Label(hl,text='  ●',bg=C['surface2'],fg='#52d9ff',font=('Consolas',9)).pack(side='left')
        tk.Label(hl,text=' Código Fuente  —  TextBox1',bg=C['surface2'],fg=C['txt'],
                 font=('Consolas',9,'bold')).pack(side='left')
        self.lbl_file=tk.Label(hl,text='sin_titulo.c',bg=C['surface2'],fg=C['muted'],font=('Consolas',8))
        self.lbl_file.pack(side='right',padx=8)
        ew=tk.Frame(lf,bg=C['editor_bg'])
        ew.pack(fill='both',expand=True)
        self.ln=tk.Text(ew,width=4,state='disabled',bg='#06101e',fg=C['muted'],
                        font=('Consolas',12),relief='flat',padx=4,pady=8,
                        cursor='arrow',selectbackground='#06101e')
        self.ln.pack(side='left',fill='y')
        tk.Frame(ew,bg=C['border'],width=1).pack(side='left',fill='y')
        self.TextBox1=tk.Text(ew,bg=C['editor_bg'],fg=C['txt'],insertbackground='#52d9ff',
                              font=('Consolas',12),relief='flat',padx=10,pady=8,
                              undo=True,wrap='none',selectbackground=C['sel'])
        self._sb1y=tk.Scrollbar(ew,orient='vertical',command=self._sync_scroll,
                                bg=C['surface'],troughcolor=C['bg'])
        sb1x=tk.Scrollbar(lf,orient='horizontal',command=self.TextBox1.xview,
                          bg=C['surface'],troughcolor=C['bg'])
        self.TextBox1.configure(yscrollcommand=self._on_editor_scroll,xscrollcommand=sb1x.set)
        sb1x.pack(side='bottom',fill='x')
        self._sb1y.pack(side='right',fill='y')
        self.TextBox1.pack(side='left',fill='both',expand=True)
        for tipo,color in self.SYNTAX_TAGS.items():
            self.TextBox1.tag_config(f'hl_{tipo}',foreground=color)
        self.TextBox1.tag_config('hl_COMENTARIO_LINEA',foreground='#546e7a',font=('Consolas',12,'italic'))
        self.TextBox1.tag_config('hl_COMENTARIO_BLOQUE',foreground='#546e7a',font=('Consolas',12,'italic'))
        self.TextBox1.tag_config('hl_ERROR_NEGATIVO_CERO',foreground='#ff5f72',
                                 underline=True)   # subrayado para destacarlo
        self.TextBox1.bind('<KeyRelease>',self._on_key)
        self.TextBox1.bind('<MouseWheel>',self._update_ln)
        self.TextBox1.bind('<Button-4>',  self._update_ln)
        self.TextBox1.bind('<Button-5>',  self._update_ln)
        self.TextBox1.bind('<ButtonRelease-1>',self._update_cursor)
        self._update_ln()

        # ── PANEL DERECHO: Notebook ───────────────────────
        rf=tk.Frame(paned,bg=C['bg'])
        paned.add(rf,minsize=500,stretch='always')
        tk.Frame(rf,bg=C['border'],height=1).pack(fill='x')

        style=ttk.Style(self)
        style.theme_use('default')
        style.configure('MC.TNotebook',background=C['surface'],borderwidth=0)
        style.configure('MC.TNotebook.Tab',background=C['btn_bg'],foreground=C['muted'],
                        font=('Consolas',9),padding=[12,4],borderwidth=0)
        style.map('MC.TNotebook.Tab',
                  background=[('selected',C['surface2'])],
                  foreground=[('selected',C['accent'])])

        self.nb=ttk.Notebook(rf,style='MC.TNotebook')
        self.nb.pack(fill='both',expand=True)

        # ── PESTAÑA 1: Tokens ─────────────────────────────
        tab_tok=tk.Frame(self.nb,bg=C['bg'])
        self.nb.add(tab_tok,text='  Tokens  ')
        hr=tk.Frame(tab_tok,bg=C['surface2'],pady=4)
        hr.pack(fill='x')
        tk.Label(hr,text='  ●',bg=C['surface2'],fg=C['tok_num'],font=('Consolas',9)).pack(side='left')
        tk.Label(hr,text=' Lista de Tokens  —  TextBox2',bg=C['surface2'],fg=C['txt'],
                 font=('Consolas',9,'bold')).pack(side='left')
        self.lbl_count=tk.Label(hr,text='',bg=C['surface2'],fg=C['muted'],font=('Consolas',8))
        self.lbl_count.pack(side='right',padx=8)
        ch=tk.Frame(tab_tok,bg=C['surface'],pady=3)
        ch.pack(fill='x')
        for t,w in [('Línea',6),('Col',5),('Lexema',20),('Token',7),('Tipo',22)]:
            tk.Label(ch,text=t,bg=C['surface'],fg=C['hdr'],font=('Consolas',8,'bold'),
                     width=w,anchor='w').pack(side='left',padx=(6,0))
        tk.Frame(tab_tok,bg=C['border'],height=1).pack(fill='x')
        ley=tk.Frame(tab_tok,bg=C['surface2'],pady=2)
        ley.pack(fill='x')
        for t,color in [('■ RESERVADA',C['tok_reserv']),('■ DIRECTIVA',C['tok_directiva']),
                        ('■ FUNC LIB',C['tok_func']),('■ NÚMERO',C['tok_num']),
                        ('■ CADENA',C['tok_string']),('■ COMENTARIO',C['tok_comment']),
                        ('■ ERROR',C['tok_err']),('■ -0 ERROR',C['tok_err'])]:
            tk.Label(ley,text=t,bg=C['surface2'],fg=color,font=('Consolas',7)).pack(side='left',padx=4)
        tk.Frame(tab_tok,bg=C['border'],height=1).pack(fill='x')
        t2w=tk.Frame(tab_tok,bg=C['editor_bg'])
        t2w.pack(fill='both',expand=True)
        self.TextBox2=tk.Text(t2w,bg=C['editor_bg'],fg=C['txt'],font=('Consolas',11),
                              relief='flat',padx=8,pady=4,state='disabled',wrap='none',
                              selectbackground=C['sel'],cursor='hand2')
        sb2y=tk.Scrollbar(t2w,orient='vertical',command=self.TextBox2.yview,
                          bg=C['surface'],troughcolor=C['bg'])
        sb2x=tk.Scrollbar(tab_tok,orient='horizontal',command=self.TextBox2.xview,
                          bg=C['surface'],troughcolor=C['bg'])
        self.TextBox2.configure(yscrollcommand=sb2y.set,xscrollcommand=sb2x.set)
        sb2x.pack(side='bottom',fill='x')
        sb2y.pack(side='right',fill='y')
        self.TextBox2.pack(fill='both',expand=True)
        self.TextBox2.bind('<Button-1>', self._on_token_click)
        for tag,fg in [('linea',C['tok_linea']),('col_t',C['muted']),
                       ('lexema',C['tok_lex']),('reserv',C['tok_reserv']),
                       ('id',C['tok_id']),('sym',C['tok_sym']),('num',C['tok_num']),
                       ('err',C['tok_err']),('directiva',C['tok_directiva']),
                       ('func',C['tok_func']),('comment',C['tok_comment']),
                       ('string',C['tok_string']),('char_lit',C['tok_char']),
                       ('hex',C['tok_hex']),('cpp',C['tok_cpp'])]:
            self.TextBox2.tag_config(tag,foreground=fg)
        # Tag especial para -0 en la lista de tokens (subrayado + rojo)
        self.TextBox2.tag_config('neg_cero', foreground=C['tok_err'], underline=True)

        # ── PESTAÑA 2: PROCESO de Reconocimiento ──────────
        tab_proc=tk.Frame(self.nb,bg=C['bg'])
        self.nb.add(tab_proc,text='  Proceso de Reconocimiento  ')
        hp=tk.Frame(tab_proc,bg=C['surface2'],pady=4)
        hp.pack(fill='x')
        tk.Label(hp,text='  ●',bg=C['surface2'],fg=C['proc_result'],font=('Consolas',9)).pack(side='left')
        tk.Label(hp,text=' Proceso detallado — haz clic en cualquier token de la lista',
                 bg=C['surface2'],fg=C['txt'],font=('Consolas',9,'bold')).pack(side='left')
        tk.Frame(tab_proc,bg=C['border'],height=1).pack(fill='x')
        self.proc_text=tk.Text(tab_proc,bg=C['editor_bg'],fg=C['txt'],
                               font=('Consolas',11),relief='flat',padx=16,pady=12,
                               state='disabled',wrap='word')
        sbpy=tk.Scrollbar(tab_proc,orient='vertical',command=self.proc_text.yview,
                          bg=C['surface'],troughcolor=C['bg'])
        self.proc_text.configure(yscrollcommand=sbpy.set)
        sbpy.pack(side='right',fill='y')
        self.proc_text.pack(fill='both',expand=True)
        self.proc_text.tag_config('titulo',  foreground=C['accent'],    font=('Consolas',12,'bold'))
        self.proc_text.tag_config('paso',    foreground=C['txt'],       font=('Consolas',11))
        self.proc_text.tag_config('arrow',   foreground=C['proc_arrow'],font=('Consolas',11,'bold'))
        self.proc_text.tag_config('ok',      foreground=C['proc_ok'],   font=('Consolas',11,'bold'))
        self.proc_text.tag_config('err_tag', foreground=C['proc_err'],  font=('Consolas',11,'bold'))
        self.proc_text.tag_config('result',  foreground=C['proc_result'],font=('Consolas',12,'bold'))
        self.proc_text.tag_config('dim',     foreground=C['muted'])

        # ── PESTAÑA 3: Tabla de Símbolos ──────────────────
        tab_sim=tk.Frame(self.nb,bg=C['bg'])
        self.nb.add(tab_sim,text='  Tabla de Símbolos  ')
        hs=tk.Frame(tab_sim,bg=C['surface2'],pady=4)
        hs.pack(fill='x')
        tk.Label(hs,text='  ●',bg=C['surface2'],fg=C['tok_directiva'],font=('Consolas',9)).pack(side='left')
        tk.Label(hs,text=' Tabla de Símbolos del Lenguaje',bg=C['surface2'],fg=C['txt'],
                 font=('Consolas',9,'bold')).pack(side='left')
        tk.Frame(tab_sim,bg=C['border'],height=1).pack(fill='x')
        ff=tk.Frame(tab_sim,bg=C['surface'],pady=4)
        ff.pack(fill='x')
        tk.Label(ff,text='  Buscar:',bg=C['surface'],fg=C['muted'],font=('Consolas',9)).pack(side='left')
        self._filtro_var=tk.StringVar()
        self._filtro_var.trace_add('write',self._filtrar_tabla)
        tk.Entry(ff,textvariable=self._filtro_var,bg=C['editor_bg'],fg=C['txt'],
                 insertbackground=C['txt'],font=('Consolas',10),relief='flat',
                 width=20).pack(side='left',padx=6)
        self._filtro_cat=tk.StringVar(value='TODOS')
        cats=['TODOS','RESERVADA','RESERVADA_CPP','DIRECTIVA',
              'FUNC_STDIO','FUNC_STDLIB','FUNC_STRING',
              'FUNC_CONIO','FUNC_MATH','SIMBOLO']
        om=tk.OptionMenu(ff,self._filtro_cat,*cats,command=lambda _: self._filtrar_tabla())
        om.config(bg=C['btn_bg'],fg=C['btn_fg'],activebackground=C['border'],
                  font=('Consolas',9),relief='flat',highlightthickness=0)
        om['menu'].config(bg=C['surface'],fg=C['txt'])
        om.pack(side='left',padx=4)
        tk.Frame(tab_sim,bg=C['border'],height=1).pack(fill='x')
        tf=tk.Frame(tab_sim,bg=C['editor_bg'])
        tf.pack(fill='both',expand=True)
        style.configure('Sym.Treeview',background=C['editor_bg'],foreground=C['txt'],
                        fieldbackground=C['editor_bg'],font=('Consolas',10),rowheight=22)
        style.configure('Sym.Treeview.Heading',background=C['surface'],foreground=C['hdr'],
                        font=('Consolas',9,'bold'),relief='flat')
        style.map('Sym.Treeview',background=[('selected',C['sel'])])
        self.tree=ttk.Treeview(tf,columns=('lexema','token','categoria'),
                               show='headings',style='Sym.Treeview')
        self.tree.heading('lexema',    text='Lexema',    anchor='w')
        self.tree.heading('token',     text='Token',     anchor='w')
        self.tree.heading('categoria', text='Categoría', anchor='w')
        self.tree.column('lexema',    width=200,anchor='w')
        self.tree.column('token',     width=80, anchor='w')
        self.tree.column('categoria', width=200,anchor='w')
        sbty=tk.Scrollbar(tf,orient='vertical',command=self.tree.yview,
                          bg=C['surface'],troughcolor=C['bg'])
        self.tree.configure(yscrollcommand=sbty.set)
        sbty.pack(side='right',fill='y')
        self.tree.pack(fill='both',expand=True)
        self._tabla_completa=[] 
        self._poblar_tabla_simbolos()

        # BARRA DE ESTADO
        tk.Frame(self,bg=C['border'],height=1).pack(fill='x',side='bottom')
        sbf=tk.Frame(self,bg=C['surface'],pady=3)
        sbf.pack(fill='x',side='bottom')
        self.lbl_archivo=tk.Label(sbf,text='Sin título',bg=C['surface'],fg=C['muted'],font=('Consolas',8))
        self.lbl_archivo.pack(side='left',padx=10)
        self.lbl_cursor=tk.Label(sbf,text='Ln 1  Col 1',bg=C['surface'],fg=C['muted'],font=('Consolas',8))
        self.lbl_cursor.pack(side='left',padx=20)
        self.lbl_status=tk.Label(sbf,text='● Listo',bg=C['surface'],fg=C['accent'],font=('Consolas',8,'bold'))
        self.lbl_status.pack(side='right',padx=10)

    # ── Helpers ─────────────────────────────────────────────
    def _sync_scroll(self,*a):
        self.TextBox1.yview(*a); self.ln.yview(*a)

    def _on_editor_scroll(self,*a):
        self._sb1y.set(*a); self.ln.yview_moveto(a[0])

    def _update_ln(self,event=None):
        self.ln.config(state='normal')
        self.ln.delete('1.0','end')
        n=int(self.TextBox1.index('end-1c').split('.')[0])
        self.ln.insert('1.0','\n'.join(str(i) for i in range(1,n+1)))
        self.ln.config(state='disabled')

    def _update_cursor(self,event=None):
        idx=self.TextBox1.index('insert')
        ln,col=idx.split('.')
        self.lbl_cursor.config(text=f'Ln {ln}  Col {int(col)+1}')

    def _on_key(self,event=None):
        self._update_ln(event); self._update_cursor(event)
        if self._hl_after: self.after_cancel(self._hl_after)
        self._hl_after=self.after(400,self._resaltar_editor)

    def _resaltar_editor(self):
        codigo=self.TextBox1.get('1.0','end-1c')
        for tipo in self.SYNTAX_TAGS:
            self.TextBox1.tag_remove(f'hl_{tipo}','1.0','end')
        AL=AnalizadorLexico()
        tokens=AL.AnalisisLexico(codigo)
        for lin,col,lex,tok,tipo,_ in tokens:
            if tipo=='IDENTIFICADOR': continue
            tag=f'hl_{tipo}'
            start=f'{lin}.{col-1}'
            end=f'{lin}.{col-1+len(lex)}'
            try: self.TextBox1.tag_add(tag,start,end)
            except: pass

    def _on_token_click(self, event):
        if not self._tokens: return
        idx = self.TextBox2.index(f'@{event.x},{event.y}')
        fila = int(idx.split('.')[0]) - 1
        if 0 <= fila < len(self._tokens):
            tok = self._tokens[fila]
            self._mostrar_proceso(tok)
            self.nb.select(1)

    def _mostrar_proceso(self, tok):
        lin, col, lex, token_id, tipo, pasos = tok
        C = self.C
        self.proc_text.config(state='normal')
        self.proc_text.delete('1.0','end')

        def w(t, tag='paso'): self.proc_text.insert('end', t, tag)

        w(f'  ══════════════════════════════════════════════\n', 'dim')
        w(f'  PROCESO DE RECONOCIMIENTO\n', 'titulo')
        w(f'  ══════════════════════════════════════════════\n\n', 'dim')

        w(f'  Lexema   : ', 'dim'); w(f'"{lex}"\n', 'result')
        w(f'  Token ID : ', 'dim'); w(f'{token_id}\n', 'ok' if token_id != 'ERR' else 'err_tag')
        w(f'  Tipo     : ', 'dim')
        es_err = 'ERROR' in tipo or 'NO ENCONTRADO' in tipo
        w(f'{tipo}\n', 'err_tag' if es_err else 'ok')
        w(f'  Posición : ', 'dim'); w(f'Línea {lin}  Columna {col}\n\n', 'paso')

        w(f'  PASOS DEL AUTÓMATA:\n', 'paso')
        w(f'  ──────────────────────────────────────────────\n', 'dim')

        for paso in pasos:
            if paso.strip().startswith('→'):
                w(f'  {paso}\n', 'arrow')
            elif 'RESULTADO' in paso:
                w(f'\n  {paso}\n', 'result')
            elif 'ERROR' in paso or '⚠' in paso:
                w(f'  {paso}\n', 'err_tag')
            elif '✔' in paso:
                w(f'  {paso}\n', 'ok')
            else:
                w(f'  {paso}\n', 'paso')

        w(f'\n  ══════════════════════════════════════════════\n', 'dim')
        self.proc_text.config(state='disabled')

    def _poblar_tabla_simbolos(self):
        UL=UnidadesLexicas(); filas=[]
        for lex,tok in sorted(UL.Palabra.items(),key=lambda x: x[1]):
            if   1<=tok<=32:    cat='RESERVADA'
            elif 33<=tok<=48:   cat='RESERVADA_CPP'
            elif 50<=tok<=59:   cat='DIRECTIVA'
            elif 60<=tok<=69:   cat='FUNC_STDIO'
            elif 70<=tok<=74:   cat='FUNC_STDLIB'
            elif 121<=tok<=130: cat='FUNC_STRING'
            elif 131<=tok<=139: cat='FUNC_CONIO'
            elif 140<=tok<=149: cat='FUNC_MATH'
            elif 150<=tok<=159: cat='RESERVADA_CPP'
            else:               cat='OTRO'
            filas.append((lex,tok,cat))
        for lex,tok in sorted(UL.Simbolo.items(),key=lambda x: x[1]):
            filas.append((lex,tok,'SIMBOLO'))
        self._tabla_completa=filas
        self._llenar_tree(filas)

    def _llenar_tree(self,filas):
        for item in self.tree.get_children(): self.tree.delete(item)
        C=self.C
        colores={'RESERVADA':C['tok_reserv'],'RESERVADA_CPP':C['tok_cpp'],
                 'DIRECTIVA':C['tok_directiva'],'FUNC_STDIO':C['tok_func'],
                 'FUNC_STDLIB':C['tok_func'],'FUNC_STRING':C['tok_func'],
                 'FUNC_CONIO':C['tok_func'],'FUNC_MATH':C['tok_func'],
                 'SIMBOLO':C['tok_sym']}
        for lex,tok,cat in filas:
            iid=self.tree.insert('','end',values=(lex,tok,cat))
            fg=colores.get(cat,C['txt'])
            self.tree.tag_configure(cat,foreground=fg)
            self.tree.item(iid,tags=(cat,))

    def _filtrar_tabla(self,*_):
        texto=self._filtro_var.get().lower(); cat=self._filtro_cat.get()
        filas=[(l,t,c) for l,t,c in self._tabla_completa
               if texto in l.lower() and (cat=='TODOS' or c==cat)]
        self._llenar_tree(filas)

    def OpcNuevo_Click(self,*_):
        if self.TextBox1.get('1.0','end-1c').strip():
            r=messagebox.askyesnocancel('Nuevo','¿Guardar cambios?')
            if r is True: self.OpcGuardar_Click()
            elif r is None: return
        self.TextBox1.delete('1.0','end')
        self._limpiar_tokens()
        self.Archivo=''
        self.title('MicroC  —  AUTOMATAS  |  FASE II')
        self.lbl_file.config(text='sin_titulo.c')
        self.lbl_archivo.config(text='Sin título')
        self.lbl_status.config(text='● Nuevo archivo',fg=self.C['accent'])
        self._update_ln()

    def OpcAbrir_Click(self,*_):
        ruta=filedialog.askopenfilename(title='Abrir',
            filetypes=[('C/C++','*.c *.cpp *.h *.hpp'),('Texto','*.txt'),('Todos','*.*')])
        if not ruta: return
        try:
            with open(ruta,'r',encoding='utf-8',errors='replace') as f: cont=f.read()
            self.TextBox1.delete('1.0','end')
            self.TextBox1.insert('1.0',cont)
            self._limpiar_tokens()
            self.Archivo=ruta; nom=os.path.basename(ruta)
            self.title(f'MicroC  —  {nom}  |  FASE I + II + EXTRAS')
            self.lbl_file.config(text=nom)
            self.lbl_archivo.config(text=ruta)
            self.lbl_status.config(text=f'● Abierto: {nom}',fg=self.C['accent'])
            self._update_ln()
            self.after(100,self._resaltar_editor)
        except Exception as ex: messagebox.showerror('Error',str(ex))

    def OpcGuardar_Click(self,*_):
        if not self.Archivo: self.OpcGuardarComo_Click(); return
        self._guardar(self.Archivo)

    def OpcGuardarComo_Click(self,*_):
        ruta=filedialog.asksaveasfilename(title='Guardar Como',defaultextension='.c',
            filetypes=[('C','*.c'),('C++','*.cpp'),('Texto','*.txt'),('Todos','*.*')])
        if ruta: self.Archivo=ruta; self._guardar(ruta)

    def _guardar(self,ruta):
        try:
            with open(ruta,'w',encoding='utf-8') as f:
                f.write(self.TextBox1.get('1.0','end-1c'))
            nom=os.path.basename(ruta)
            self.title(f'MicroC  —  {nom}  |  FASE II')
            self.lbl_file.config(text=nom); self.lbl_archivo.config(text=ruta)
            self.lbl_status.config(text=f'● Guardado: {nom}',fg=self.C['accent'])
        except Exception as ex: messagebox.showerror('Error',str(ex))

    def OpcSalir_Click(self,*_):
        if messagebox.askokcancel('Salir','¿Cerrar MicroC?'): self.destroy()

    def compilarToolStripMenuItem_Click(self,*_):
        Archivo=self.TextBox1.get('1.0','end-1c')
        if not Archivo.strip():
            messagebox.showinfo('MicroC','El editor está vacío.'); return
        self.lbl_status.config(text='● Compilando…',fg=self.C['hdr'])
        self.update()
        AL=AnalizadorLexico()
        ListToken=AL.AnalisisLexico(Archivo)
        self._tokens=ListToken
        self._escribir_tokens(ListToken)
        self._resaltar_editor()
        total=len(ListToken)
        errores=sum(1 for t in ListToken if 'ERROR' in t[4] or 'NO ENCONTRADO' in t[4])
        self.lbl_count.config(text=f'Tokens: {total}  Errores: {errores}')
        if errores:
            self.lbl_status.config(text=f'● {errores} error(es)',fg=self.C['tok_err'])
        else:
            self.lbl_status.config(text='● Compilación exitosa',fg=self.C['tok_num'])
        self.nb.select(0)

    def _escribir_tokens(self, ListToken):
        self.TextBox2.config(state='normal')
        self.TextBox2.delete('1.0','end')
        for lin,col,lex,tok,tipo,_ in ListToken:
            self.TextBox2.insert('end',f'  {str(lin):<6}','linea')
            self.TextBox2.insert('end',f'{str(col):<5}','col_t')
            self.TextBox2.insert('end',f'{lex:<22}','lexema')
            self.TextBox2.insert('end',f'{str(tok):<9}','sym')
            if   tipo=='RESERVADA':                                tag='reserv'
            elif tipo=='RESERVADA_CPP':                            tag='cpp'
            elif tipo=='IDENTIFICADOR':                            tag='id'
            elif tipo in ('ENTERO','REAL'):                        tag='num'
            elif tipo in ('HEXADECIMAL','OCTAL','BINARIO'):        tag='hex'
            elif tipo in ('DIRECTIVA','ARG_DIRECTIVA'):            tag='directiva'
            elif tipo in ('FUNC_STDIO','FUNC_STDLIB','FUNC_STRING',
                          'FUNC_CONIO','FUNC_MATH'):               tag='func'
            elif tipo in ('COMENTARIO_LINEA','COMENTARIO_BLOQUE'): tag='comment'
            elif tipo in ('CADENA','CARACTER'):                    tag='string'
            elif tipo == 'ERROR_NEGATIVO_CERO':                    tag='neg_cero'
            elif 'ERROR' in tipo or 'NO ENCONTRADO' in tipo:      tag='err'
            else:                                                  tag='sym'
            self.TextBox2.insert('end',f'{tipo}\n',tag)
        self.TextBox2.config(state='disabled')

    def _limpiar_tokens(self,*_):
        for w in [self.TextBox2,self.proc_text]:
            w.config(state='normal'); w.delete('1.0','end'); w.config(state='disabled')
        self._tokens=[]
        self.lbl_count.config(text='')
        self.lbl_status.config(text='● Listo',fg=self.C['accent'])
        for tipo in self.SYNTAX_TAGS:
            self.TextBox1.tag_remove(f'hl_{tipo}','1.0','end')

    def _acerca(self):
        messagebox.showinfo('Acerca de MicroC',
            'MicroC — Analizador Léxico\n'
            'AUTOMATAS Y LENGUAJES  |  2026\n'
            'Semestre V  —  Ing. Sistemas\n'
            'Catedrático: Ing. Baudilio Boteo\n\n'
            '✔ FASE I  — Tokens, símbolos, identificadores\n'
            '✔ FASE II — Directivas, librerías, números,\n'
            '            comentarios, cadenas, caracteres\n\n'
            '✔ EXTRAS:\n'
            '   · Resaltado de sintaxis en tiempo real\n'
            '   · PROCESO de reconocimiento paso a paso\n'
            '     (haz clic en cualquier token para verlo)\n'
            '   · Tabla de Símbolos con búsqueda y filtro\n'
            '   · Columna de posición por token\n'
            '   · Indicador Línea/Columna del cursor\n\n'
            '✔ ERROR ESPECIAL:\n'
            '   · "-0" reconocido como ERROR léxico\n'
            '     (negativo de cero no tiene sentido)\n'
            '   · Se acepta: -1, -0.5, -0x1F  (válidos)\n'
            '   · Se rechaza: -0, -0u, -0l    (error)\n\n')


if __name__ == '__main__':
    app = frmEditor()
    app.mainloop()
