    # Alchemy.js is a graph drawing application for the web.
    # Copyright (C) 2014  GraphAlchemist, Inc.

    # This program is free software: you can redistribute it and/or modify
    # it under the terms of the GNU Affero General Public License as published by
    # the Free Software Foundation, either version 3 of the License, or
    # (at your option) any later version.

    # This program is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    # GNU Affero General Public License for more details.

    # You should have received a copy of the GNU Affero General Public License
    # along with this program.  If not, see <http://www.gnu.org/licenses/>.


    # make js array method called ._state
    # @a.set.  nest set inside of get

    Alchemy::get = (instance) ->
        a: instance
        _el: []
        _elType: null
        _makeChain: (inp)->
            returnedGet = @
            returnedGet.__proto__ = [].__proto__
            returnedGet.pop() while returnedGet.length
            _.each inp, (e)-> returnedGet.push(e)
            returnedGet

        # returns one or more nodes as an array
        nodes: (id, ids...) ->
                    if id?
                        allIDs = _.map arguments, (arg) -> String(arg)
                        a = @.a
                        nodeList = do (a) ->
                            # All passed ids with artificially enforced type safety
                            _.filter a._nodes, (val, key)->
                                val if _.contains allIDs, key
                    @_elType = "node"
                    @_el = nodeList
                    @_makeChain nodeList

        # returns one or more edges as an array
        edges: (id, ids...) ->
            if id?
                allIDs = _.map arguments, (arg) -> String(arg)
                a = @.a
                edgeList = do (a) ->
                    # All passed ids with artificially enforced type safety
                    _.flatten _.filter a._edges, (val, key)->
                        val if _.contains allIDs, key
            @_elType = "edge"
            @_el = edgeList
            @_makeChain edgeList

        all: ->
            a = @a
            elType = @_elType
            @_el = do (elType)->
                switch elType
                    when "node" then return _.values a._nodes
                    when "edge" then return _.flatten _.map a._edges, (e)-> e
            @_makeChain @_el

        elState: (state) ->
            elList = _.filter @_el, (e)-> e._state is state
            @_el = elList
            @_makeChain elList

        state: (key) -> if @a.state.key? then @a.state.key

        type: (type) ->
            elList = _.filter @_el, (e) -> e._nodeType is type or e._edgeType is type
            @_el = elList
            @_makeChain elList

        activeNodes: () ->
            _.filter @a._nodes, (node) -> node if node._state is "active"

        activeEdges: ->
            _.filter @a.get.allEdges(), (edge) -> edge if edge._state is "active"
        
        state: (key) -> if @a.state.key? then @a.state.key

        clusters: ->
            clusterMap = @a.layout._clustering.clusterMap
            nodesByCluster = {}
            _.each clusterMap, (key, value) ->
                nodesByCluster[value] = _.select @a.get.allNodes(), (node) ->
                    node.getProperties()[@a.conf.clusterKey] is value
            nodesByCluster

        clusterColours: ->
            clusterMap = @a.layout._clustering.clusterMap
            clusterColoursObject = {}
            _.each clusterMap, (key, value) ->
               clusterColoursObject[value] = @a.conf.clusterColours[key % @a.conf.clusterColours.length]
            clusterColoursObject

        ###### ALL METHODS BELOW THIS POINT WILL BE DEPRECATED UPON 1.0 ######
        allEdges: ->
            _.flatten _.map(@a._edges, (edgeArray) -> e for e in edgeArray)

        allNodes: (type) ->
            if type?
                _.filter @a._nodes, (n) -> n if n._nodeType is type
            else
                _.map @a._nodes, (n) -> n

        getNodes: (id, ids...)->
            a = @a
            ids.push(id)
            _.map ids, (id)-> a._nodes[id]

        getEdges: (id=null, target=null)->
            a = @a
            if id? and target?
                edge_id = "#{id}-#{target}"
                @a._edges[edge_id]
            else if id? and not target?
                @a._nodes[id]._adjacentEdges