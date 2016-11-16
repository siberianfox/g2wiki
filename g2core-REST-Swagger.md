
```yaml
#
# Test swagger for g2core-server
#
# HTTP status codes - https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
# 
swagger: '2.0'
info:
  version: '0.99.0'
  title: g2core REST API
  description: 
    The g2core REST API is a minimalist API that mostly just passes keys through to the g2core firmware and returns the firmware responses and some status information. The resource model also includes provisions for retrieving status, launching long running operations, and launching and managing CNC jobs.
    
    
    See top-level decription of the resource model
    https://github.com/synthetos/g2/wiki/g2core-REST-Resources
    
    
    Reference for g2core JSON keys and semantics
    https://github.com/synthetos/g2/wiki/Configuring-Version-0.99
  termsOfService: http://helloreverb.com/terms/
  contact:
    name: Synthetos.com
    email: alden.hart@synthetos.com #, rob.giseburt@synthetos.com, riley.porter@synthetos.com
    url: https://github.com/synthetos/g2/wiki/g2core-REST-Resources
  license:
    name: MIT
    url: http://opensource.org/licenses/MIT
host: 127.0.0.0
basePath: /api
schemes:
  - http
consumes:
  - application/json
produces:
  - application/json
paths:

  /info:
    get:
      description: Return machine identification and other information
      operationId: getInfo
      produces:
        - application/json
      responses:
        '200':
          description: getInfo response
          schema:
            required:
              - fv
              - fb
              - fbs
              - fbc
              - hp
              - hv
              - id
            properties:
              fv: 
                description: firmware version
                type: number
              fb:
                description: firmware build
                type: number
              fbs:
                description: firmware build string
                type: string
              fbc:
                description: firmware build configuration
                type: string
              hp:
                description: hardware platform
                type: string
              hv:
                description: hardware version
                type: string
              id:
                description: machine ID
                type: string
              msg:
                description: startup message
                type: string
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'

  /machine:
    get:
      description: Return or inspect machine resources. Omit body to get full machine.
      operationId: getMachineByJSON
      produces:
        - application/json
      parameters:
        - name: key
          in: body
          description: JSON describing values to get. Omit to get full machine.
          required: false
          schema:
            $ref: '#/definitions/jsonObject'
        - name: nodes_only
          in: query
          description: return top-level objects only
          required: false
          type: boolean
#        - name: return_URI
#          in: query
#          description: return nodes as URIs
#          required: false
#          type: boolean
      responses:
        '200':
          description: getMachineByJSON response
          schema:
            $ref: '#/definitions/genericResponse'   # only need generic response for URI return, otherwise jsonResponse
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'
    put:
      description: Update machine resources by path 
      operationId: setMachineByJSON
      parameters:
        - name: key
          in: body
          description: JSON describing values to set
          required: false
          schema:
            $ref: '#/definitions/jsonObject'
      responses:
        '200':
          description: setKey response
          schema:
            $ref: '#/definitions/genericResponse'
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'

  /machine/{path}:
    get:
      description: Return or inspect machine resources by path 
      operationId: getMachineByPath
      produces:
        - application/json
      parameters:
        - name: path
          in: path
          description: path to object or attribute to get
          required: true
          type: string
        - name: nodes_only
          in: query
          description: return top-level objects only
          required: false
          type: boolean
#        - name: return_URI
#          in: query
#          description: return nodes as URIs
#          required: false
#          type: boolean
      responses:
        '200':
          description: getMachineByPath response
          schema:
            $ref: '#/definitions/genericResponse'
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'
    put:
      description: Update machine resources by path 
      operationId: setMachineByPath
      parameters:
        - name: path
          in: path
          description: path to object or attribute to set
          required: true
          type: string
        - name: value
          in: body
          description: JSON to set one or more values
          required: true
          schema:
            $ref: '#/definitions/jsonObject'
      responses:
        '200':
          description: setKey response
          schema:
            $ref: '#/definitions/genericResponse'
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'

  /status_report:
    get:
      description: Return a g2core status report. (1) Call with no filter to retrieve complete status report, (2) Call with a JSON filter with null attribute values to retrieve specific attributes from a status report, (3) Call with a JSON filter with non-null attribute values to return only changed values. 
      
      
        See https://github.com/synthetos/g2_private/wiki/g2core-REST-Resources#status_report-resource
      operationId: getState
      produces:
        - application/json
      parameters:
        - name: filter
          in: body
          description: return status report (SR)
          required: false
          schema:
            $ref: '#/definitions/jsonObject'
      responses:
        '200':
          description: getStatus response
          schema:
            $ref: '#/definitions/genericResponse'
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'
            
  /operation:
    post:
      description: Start a new operation context
      operationId: newOperation
      produces:
        - application/json
      parameters:
        - name: op
          in: body
          description: Start a new operation
          required: true
          schema:
            $ref: '#/definitions/operationRequest'
      responses:
        '201':          # 201 RESOURCE HAS BEEN CREATED
          description: operation created
          schema:
            type: integer
            format: uint32
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'

  /operation/{id}:
    get:
      description: Return operation context by operation ID
      operationId: getOperation
      produces:
        - application/json
      parameters:
        - name: id
          in: path
          description: id of operation to retrieve
          required: true
          type: string
      responses:
        '200':
          description: getStatus response
          schema:
            $ref: '#/definitions/operationResponse'
        default:
          description: unexpected error
          schema:
            $ref: '#/definitions/genericError'

definitions:

  jsonObject:           # describes an arbitrary JSON object
    type: object

  responseStatus:     # Note, this object breaks from the g2core naming conventions
    type: object
    properties:
      code:
        description: g2core status_code (error.h)
        type: integer
        format: uint8
      description:
        description: g2core Status message strings (error.h)
        type: string
      additional_description: 
        description: g2core 'msg' additional text
        type: string

  genericResponse:
    type: object
    required:
      - value           # response value JSON object
    properties:
      value:
        type:
          - object
          - number
          - string
          - boolean
          - "null"
      msg:              # message string, error or exception text if present in response
        type: string
      status:
        $ref: '#/definitions/responseStatus'

  genericError:
    type: object
    properties:
      status:
        $ref: '#/definitions/responseStatus'

  operationRequest:
    type: object
    required:
      - optype
      - value           # text or JSON of request
    properties:
      optype:
        type: string
      value:
        type: string

  operationResponse:
    type: object
    required:
      - opstate
      - value           # JSON response
    properties:
      id:
        type: integer
        format: uint32
      opstate:          # state of operation, e.g. pend, run, done, error
        type: string
      value:
        $ref: '#/definitions/jsonObject'
      status:
        $ref: '#/definitions/responseStatus'

#  anyTypeValue:
#    type: object
#    required:
#      - value           # value can be any native JSON type or a JSON object  
#    properties:
#      value:
#       type:
#         - object       # JSON object
#         - number
#         - string
#         - boolean
#         - "null"       # Quotes reqwuired becuase nu.. is a reserved word

#  jsonResponse:
#    type: object
#    required:
#      - value           # response value JSON object
#    properties:
#      value:
#        $ref: '#/definitions/jsonObject'
#      msg:              # message string, error or exception text if present in response
#        type: string
#      status:
#        $ref: '#/definitions/responseStatus'

```
